---
layout: post
title: "Django week_day Field lookup"
date:   2017-08-14
tags:
  - django
---

Django's ORM offers built-in field lookups, one of which is `week_day`. So one can do `Sale.objects.get(sale_date__week_day=2)` to find out how well is the business doing on Mondays.

Wait what? How come 2 is a Monday? According to Python documentation, there are 2 possible weekday representations (identical methods are available for `datetime` objects):

- `date.weekday` considers a weekday from [0 (Monday) to 6 (Sunday)](https://docs.python.org/3/library/datetime.html#datetime.date.weekday)
- `date.isoweekday()` considers a weekday from [1 (Monday) to 7 (Sunday)](https://docs.python.org/3/library/datetime.html#datetime.date.isoweekday)

Whereas Django interprets the integer as [1 (Sunday) to 7 (Saturday)](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#week-day). Here's a short util to transform from Python to Django weekday value:

{% highlight python %}
def to_django_weekday(date):
    return (date.isoweekday() % 7) + 1
{% endhighlight %}

Databases too have different interpretations for this value: 

- MySQL and Oracle are identical to Django's weekday value - 1 (Sunday) to 7 (Saturday)
- PostgreSQL - 0 (Sunday) to 6 (Saturday)
- SQLite is identical to Python's `isoweekday()` representation  - 1 (Monday) to 7 (Sunday)

Even though the value varies between databases, it's still not exactly clear why this value is so alien to any of the Python representations. I think I will pester some people today at [DjangoCon US](https://2017.djangocon.us/).
