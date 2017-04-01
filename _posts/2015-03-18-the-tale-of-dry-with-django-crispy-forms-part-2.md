---
layout: post
title: "The tale of DRY with django-crispy-forms | Part II"
date:   2015-03-18
tags:
  - code
  - python
  - django
---

When you write a blog post, people of Internet will point out any mistakes
they find. It's a great opportunity to improve existing code and look into other
solutions. After posting [The tale of DRY with django-crispy-forms]({{ site.baseurl }}/2014/11/04/the-tale-of-dry-with-django-crispy-forms/)
I found out about [formulation](https://github.com/funkybob/formulation) -
a template-based solution for rendering forms, in contrast to crispy-forms that
builds the form structure in code.

Also [@maraujop](https://twitter.com/maraujop) -- crispy-forms developer -- made
several very helpful suggestions in the comments area, which brings us to Part II.


## Mistakes of the past

The story of Foo people can be found in this [archive]({{ site.baseurl }}/2014/11/04/the-tale-of-dry-with-django-crispy-forms/).
Their end was tragic. They knew truth was somewhere close, but they were unable
to grasp it to its fullest and failed to survive this cruel software world.

`FooForm`s had their `__init__` cluttered with the initialization
of a `FormHelper` object and the attachment of  the `SubmitCancelFormActions`.
Nothing bad with `SubmitCancelFormActions` itself, but as they grew in number
overriding the initialization started to look silly.

Decline of Foo marked the birth of powerful Bar people.

{% include figure.html src="/assets/img/dry-tale-2/laputa.jpg" caption="Laputa: Castle in the Sky (Ghibli Studio) - How I imagine the fairy tale kingdoms" %}

## Reforms

Assume that Bar model, view and template are similar to Foo. Bar people knew
they should start with the API, and so they shaped their dreams first.

{% highlight python %}
{% raw %}
# fairytale/characters/forms.py
from characters.models import Bar
from common.forms import ModelFormWithHelper
from common.helpers import SubmitCancelFormHelper


class BarForm(ModelFormWithHelper):
    """Model Bar form"""
    class Meta:
        model = Bar
        helper_class = SubmitCancelFormHelper
        helper_cancel_href = "{% url 'some_cancel_url' %}"

{% endraw %}
{% endhighlight %}

What Bar wants is to declare an alternative `FormHelper`, which is responsible
for customizing the form. Also it wants to provide helper attributes, that are different
for each form - the cancel URL, for example.

There are 2 unknown creatures here - `SubmitCancelFormHelper` and `ModelFormWithHelper`.

`SubmitCancelFormHelper` as you might have guessed is of `FormHelper` breed. It
tells you right to the face that it appends to the layout submit and cancel buttons.
See for yourself.

{% highlight python %}
{% raw %}
# fairytale/common/helpers.py
from crispy_forms.bootstrap import FormActions
from crispy_forms.helper import FormHelper, Layout
from crispy_forms.layout import Submit, HTML


class SubmitCancelFormHelper(FormHelper):
    """Custom FormHelper that appends to the layout cancel and submit
    buttons (works only with bootstrap crispy-forms pack). It expects a
    `cancel_href` attribute to be used as cancel button URL.

    Example::

        SubmitCancelFormHelper(self, cancel_href="/some/url/")
    """
    def __init__(self, *args, **kwargs):
        cancel_href = kwargs.pop('cancel_href', '#')
        super(SubmitCancelFormHelper, self).__init__(*args, **kwargs)
        self.layout.append(
            Layout(
                FormActions(
                    HTML("""<a role="button" class="btn btn-default mr4"
                            href="{0}">Cancel</a>""".format(cancel_href)),
                    Submit('save', 'Submit'),
                )
            )
        )
{% endraw %}
{% endhighlight %}

This class expects a `cancel_href` named parameter. Afterwards it takes
advantage of the power it was given by `FormHelper` and modifies the layout by
appending a button that we represent using HTML widget and a Submit widget.
`SubmitCancelFormHelper` serves us well.

What about `ModelFormWithHelper` mystery. This base class is a bit more tricky,
but still nothing out of this world.

{% highlight python %}
{% raw %}
# fairytale/common/helpers.py
from django.core.exceptions import ImproperlyConfigured
from django.forms import ModelForm


class ModelFormWithHelper(ModelForm):
    """Custom ModelForm that allows to attach a crispy-forms FormHelper class,
    that will modify in some way the rendering of the layout.

    Example::

        FooForm(ModelFormWithHelper):
            class Meta:
                model = FooModel
                helper_class = FooFormHelper
    """
    def __init__(self, *args, **kwargs):
        super(ModelFormWithHelper, self).__init__(*args, **kwargs)

        if hasattr(self.Meta, "helper_class"):
            helper_class = getattr(self.Meta, "helper_class")
            kwargs = self.get_helper_kwargs()
            self.helper = helper_class(self, **kwargs)
        else:
            raise ImproperlyConfigured(
                "{0} is missing a 'helper_class' meta attribute.".format(
                    self.__class__.__name__))

    def get_helper_kwargs(self):
        """Get all helper attributes from class Meta by stripping them of
        `helper_` part of attribute string

        :return: dict with helper kwargs
        """
        kwargs = {}
        for attr, value in self.Meta.__dict__.items():
            if attr.startswith("helper_") and attr != "helper_class":
                new_attr = attr.split("_", 1)[1]
                kwargs[new_attr] = value
        return kwargs
{% endraw %}
{% endhighlight %}

Ohh, that's a lot of code. Well, docstrings make it 1/3 of the listing. Let's
see what `ModelFormWithHelper` is trying to tell us.

In the initialization phase it checks if the class
has a `helper_class` attribute. It's mandatory to be there, otherwise there's
no point in using this class at all. If there isn't, `ModelFormWithHelper` will
throw an exception to remind you, *Hey, you forgot that attribute - helper_class!*
Otherwise, it will initialize the form helper class and attach it
to `self.helper`. But before that it extracts the helper kwargs.

Look at the dream `BarForm` class. Bar people wanted to have a custom cancel URL
and used a meta attribute `helper_cancel_href`. Method `get_helper_kwargs()` deals
with all the `helper_` attributes and strips them of that prefix to ultimately
feed them to the helper.

The class is not tied to any specific form helper, so you can create as many custom
form helpers as your kingdom needs.

That's it. Now forms will become more compact and readable due to the new meta
attributes.

