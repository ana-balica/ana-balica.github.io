---
layout: post
title: "Pagination in Django"
date: 2015-01-29
tags:
  - code
  - django
---

Isn't it delightful when things just work? That's how it is for Django pagination
feature.

Say you are writing a blog in Django. Actually seems like a silly idea. If you
want a blog, get yourself a static blog, or tumblr, or wordpress, or blogger
(is it still popular?).
Ok, maybe it's not a blog. Probably you want to display a list of books in a
paginated way. Definitely not cool to list all 2k books on a single page.

I like Django class-based views. They do for you all the hard work if you stick
to the basics. So let's start writing a CBV. Assume the model contains any kind of gibberish.

{% highlight python %}
# my_library/library/views.py
from django.views.generic import ListView

from library.models import Book


class BookListView(ListView):
    model = Book
    paginate_by = 10

{% endhighlight %}

This is basically what we ought to do to enable pagination. Just specify how many
objects we want on one page. Let's take a look
at the route and the template.

{% highlight python %}
# my_library/urls.py
from django.conf.urls import patterns, url

from library.views import BookListView

urlpatterns = patterns('',
    url(r'^books/$', BookListView.as_view(), name='book_list'),
)

{% endhighlight %}

{% highlight python %}
{% raw %}
{# my_library/templates/book_list.html #}
{# omit all the additional HTML markup and leave the bare minimum #}
{% for book in object_list %}
  {{ book.title }}
{% endfor %}
{% endraw %}
{% endhighlight %}

Now if you start your server and access the URL `127.0.0.1:8000/books/` you will
get 10 book objects listed on the first page. In order to access the second page
you need to enter `127.0.0.1:8000/books/?page=2`. But your user isn't supposed
to know that (don't even think about writing a manual for that and giving it
to the end user). That's why we will create pagination buttons.

Django `ListView` is so smart that it will add to your context some special
objects that will allow you to build those pagination buttons. In fact the
pagination we will build is reusable, so we will put in a separate snippet and
include to any template that requires pagination.

{% highlight html %}
{% raw %}
{# my_library/templates/snippets/pagination.html #}
{% if is_paginated %}
  <nav>
    <ul class="pagination">
      {% if page_obj.has_previous %}
        <li>
          <a href="?page={{ page_obj.previous_page_number }}">
            <span>Previous</span>
          </a>
        </li>
      {% else %}
        <li class="disabled">
          <a href="#">
            <span>Previous</span>
          </a>
        </li>
      {% endif %}

      {% for page in paginator.page_range %}
        <li {% if page == page_obj.number %}class="active"{% endif %}>
          <a href="?page={{ page }}">{{ page }}</a>
        </li>
      {% endfor %}

      {% if page_obj.has_next %}
        <li>
          <a href="?page={{ page_obj.next_page_number }}">
            <span>Next</span>
          </a>
        </li>
      {% else %}
        <li {% if not page_obj.has_next %}class="disabled"{% endif %}>
          <a href="#">
            <span>Next</span>
          </a>
        </li>
      {% endif %}
    </ul>
  </nav>
{% endif %}
{% endraw %}
{% endhighlight %}

Now back to our `book_list.html` template.

{% highlight python %}
{% raw %}
{# my_library/templates/book_list.html #}
{% for book in object_list %}
  {{ book.title }}
{% endfor %}

{% include 'snippets/pagination.html' %}
{% endraw %}
{% endhighlight %}

Dead simple. And this snippet will serve you well for all kinds of list views.

