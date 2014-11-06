--- 
layout: post 
title: "The tale of DRY with django-crispy-forms"
date:   2014-11-04 
---

So you have this Django app. If you are like me and don't have quite the skills
for a good UI design, you probably stick to
[bootstrap](http://getbootstrap.com/). You know you will have a bunch of forms
and you don't want to style them yourself. Let's say, you came to the idea that
[django-crispy-forms](http://django-crispy-forms.rtfd.org) fits your needs 
perfectly (it definitely fits mine). 

### How it all began

This is a story about a model *Foo*.

{% highlight python %}
# fairytale/characters/models.py
from django.db import models


class Foo(models.Model):
    """A magical creature from Foo dynasty"""
    mighty_name = models.CharField(max_length=255)
    kingdoms_count = models.PositiveIntegerField(default=0)
    email = models.EmailField()  # What? Magical creatures also have emails.
{% endhighlight %}

Model *Foo* was lonely, so it summoned a view, a template and a url.

{% highlight python %}
# fairytale/characters/views.py
from django.views.generic.edit import CreateView

from characters.models import Foo


class CreateFooView(CreateView):
    """A create view for Foo model"""
    template_name = "characters/create_foo.html"
    model = Foo
{% endhighlight %}

{% highlight html %}
{% raw %}
# fairytale/templates/characters/create_foo.html
{% extends "base.html" %}

{% load crispy_forms_tags %}
{% block content %}
  <div class="row">
    <div class="col-md-6">
      <h1>Create Foo</h1>
      <hr/>
      <div class="well">
        {% crispy form %}
      </div>
    </div>
  </div>
{% endblock %}
{% endraw %}
{% endhighlight %}

The `base.html` contents are left out, as they include standard HTML markup, 
bootstrap stylesheet include and a mandatory template block `content` (in our
specific case).

{% highlight python %}
# fairytale/fairytale/urls.py
from django.conf.urls import patterns, url

from characters.views import CreateFooView


urlpatterns = patterns(
    '',
    url(r'^foo/create/$', CreateFooView.as_view(), name="create_foo"),
)
{% endhighlight %}

And this is what we created so far:

![Foo form without submit]({{ site.baseurl }}/assets/img/dry-tale/create_form_no_submit.png)

Such cool, right?

![Doge Django meme]({{ site.baseurl }}/assets/img/dry-tale/doge-meme.jpg)

Oh, wait! Where are the buttons? How do I submit this thing? *Foo the Emperor* 
must live!

Ok, I will need a *Cancel* anchor tag that will look like a button, because I 
want to redirect to some other page, and a *Submit* input. Easy!

{% highlight python %}
{% raw %}
# fairytale/characters/forms.py
from django import forms
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit, HTML
from crispy_forms.bootstrap import FormActions

from characters.models import Foo


class FooForm(forms.ModelForm):
    """Model Foo form"""
    class Meta:
        model = Foo

    def __init__(self, *args, **kwargs):
        super(FooForm, self).__init__(*args, **kwargs)
        self.helper = FormHelper(self)
        self.helper.layout.append(
            FormActions(
                HTML("""<a role="button" class="btn btn-default"
                        href="{% url "some_cancel_url" %}">Cancel</a>"""),
                Submit('save', 'Submit'),
        ))
{% endraw %}
{% endhighlight %}

Meh, not the prettiest way to achieve what I want, but apparently this is the 
only easy and straight-forward way. Also don't forget to update the view.

{% highlight python %}
# fairytale/characters/views.py
ifrom django.views.generic.edit import CreateView

from characters.models import Foo


class CreateFooView(CreateView):
    """A create view for Foo model"""
    template_name = "characters/create_foo.html"
    form_class = FooForm  # the change is over here
    model = Foo
{% endhighlight %}

And so this is what we get:

![Foo form with submit]({{ site.baseurl }}/assets/img/dry-tale/create_form_with_submit.png)

The lack of space between the anchor tag and the input looks awful, right? This
tiny cosmetic issue was discussed in 
[issue #62](https://github.com/maraujop/django-crispy-forms/issues/62) and 
solved for groups of buttons and inputs. Unfortunately it persists in case of 
an input/button and anchor tag with `role="button"`. If it's bugging you, add 
some margin by yourself. 

### The misfortune

Life goes on and *Foo* objects seem to be doing just fine. Until a great misfortune
takes over their destiny. It doesn't seem like a big deal, since all that 
happened was the addition of two more characters: *Bar* and *Baz*. What harm 
can they potentially cause? Well, let's see.

*Bar* and *Baz* follow the same old path *Foo* did a long time ago: 
model/form/view/template/url. The form again will initialize a `FormHelper` to 
append those 2 elements. It seems like every form will need them (like duhh, how
can I get around the form without them?). And so we copy-pasty, we copy-paste.

And then the pressure of the invisible monster can be felt, crawling under the 
skin and whispering to you: "You will do this over and over again. For every
form. And when you will want to change a tiny detail, you will change it for 
every form, because you want to stay consistent. That can be the change in the
bootstrap anchor tag interface, class of the *Submit* input, text of the anchor
tag."

This is the punishment for the violation of DRY (Don't Repeat Yourself)
principle. 

It's not too late to fix the situation, until the quicksand doesn't devour
all the codebase into the abyss of "refactoring is too expensive for us". 

### Layout objects to the rescue

At first, it seems like a good idea to use 
[Composing layouts](http://django-crispy-forms.readthedocs.org/en/latest/layouts.html#composing-layouts).
We take that chunk of layout and simply store it into a variable somewhere. But 
how can we change that *Cancel* URL? Or maybe we plan to have different input
values for *submit* input for different forms. There might be several variables 
that we would like to keep flexible for a more or less fixed piece of layout. 

One of the solutions is to create our own custom `ObjectLayout`. It involves a
little bit of `django-crispy-forms` internals, but don't worry, I'm here to 
help. 

This is where we start from - 
[creating your own layout objects](http://django-crispy-forms.readthedocs.org/en/latest/layouts.html#creating-your-own-layout-objects).
As it says in the current section, we investigate the objects that live in 
`layout.py` and `bootstrap.py`. We come up with the following cure for the 
misfortune:

{% highlight python %}
# fairytale/common/crispy_forms/bootstrap.py
from crispy_forms.bootstrap import FormActions
from crispy_forms.layout import LayoutObject, Submit, HTML


class SubmitCancelFormActions(LayoutObject):
    """Custom bootstrap layout object. It wraps a Cancel anchor tag and a Submit
    input field in a <div class="form-actions">.

    Example::

        SubmitCancelFormActions(cancel_href="/some/url/")
    """
    def __init__(self, *fields, **kwargs):
        self.cancel_href = kwargs.pop('cancel_href', '#')

    def render(self, form, form_style, context):
        layout_object = FormActions(
            HTML("""<a role="button" class="btn btn-default" href="{0}">
                    Cancel</a>""".format(self.cancel_href)),
            Submit('save', 'Submit'),
        )
        return layout_object.render(form, form_style, context)
{% endhighlight %}

In the `__init__` we extract all the arguments we are interested in, for example
the URL of the *Cancel* anchor tag. The `render` method produces the HTML string
with all the nodes translated. There is no need to create our own template to be
rendered, since we can reuse `render` method provided by the `django-crispy-forms`.
It already knows how to translate all those objects we have put together, to plain
HTML. 

{% highlight python %}
{% raw %}
# fairytale/characters/forms.py
from django import forms
from crispy_forms.helper import FormHelper

from characters.models import Foo
from common.crispy_forms.bootstrap import SubmitCancelFormActions


class FooForm(forms.ModelForm):
    """Model Foo form"""
    class Meta:
        model = Foo

    def __init__(self, *args, **kwargs):
        super(FooForm, self).__init__(*args, **kwargs)
        self.helper = FormHelper(self)
        self.helper.layout.append(
            SubmitCancelFormActions(
                cancel_href="{% url 'some_cancel_url' %}") 
        ))
{% endraw %}
{% endhighlight %}

Maybe it doesn't seem like a big deal, saves us maybe 2 lines of code in the 
form and adds up lots of code some place else. As the monster warned us, the 
real danger comes with the growth of objects and forms in number. At some point 
it will become much more complicated to make any changes to those form actions. 
It is a small enhancement that might save time for future *me*. 

Alright, that looks much better. Both for *Foo*, *Bar* and *Baz*. Oh, and don't
forget about unittest, otherwise another monster will come to bite your ass. 

Let's see how can we write a unittest for our little layout object. I will 
cheat a bit and use some test tools and shortcuts created specifically for
`django-crispy-forms`. 

{% highlight python %}
{% raw %}
# fairytale/common/tests.py
from django.template import loader, Context
from django.test import TestCase
from crispy_forms.tests.forms import TestForm
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Layout

from common.forms.bootstrap import SubmitCancelFormActions


class FormLayoutObjectsTestCase(TestCase):
    def test_submit_cancel_form_actions(self):
        """Test custom SubmitCancelFormActions bootstrap layout object"""
        template = loader.get_template_from_string(u"""
            {% load crispy_forms_tags %}
            {% crispy form %}
        """)

        test_form = TestForm()
        test_form.helper = FormHelper()
        test_form.helper.layout = Layout(
            SubmitCancelFormActions()
        )

        c = Context({'form': test_form})

        html = template.render(c)
        self.assertEqual(html.count('class="form-actions'), 1)
        self.assertEqual(html.count('role="button"'), 1)
        self.assertEqual(html.count('href="#"'), 1)
        self.assertEqual(html.count('Cancel'), 1)
        self.assertEqual(html.count('Submit'), 1)

        test_form.helper.layout = Layout(
            SubmitCancelFormActions(cancel_href="/some/url/")
        )
        c = Context({'form': test_form})

        html = template.render(c)
        self.assertEqual(html.count('href="/some/url/'), 1)
{% endraw %}
{% endhighlight %}

In the unittest we render the form with the custom layout object 
`SubmitCancelFormActions` and check for some keywords if present in the rendered 
HTML.

### Happy ending

That's all folks! Keep DRY and write unittests!

-----

P.S. In case you are still in doubt, here is what [Daniel Greenfeld](http://pydanny.com/) 
says: "As one of the leads of django-crispy-forms, I approve this blog post. :-)"
