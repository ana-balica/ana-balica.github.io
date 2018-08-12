---
layout: post
title: "How to filter by DateRange with django-filters"
date:   2018-08-12
tags:
  - python
  - django
---

Django offers a few fields that are exclusive to Postgres. I recently had to use [DateRangeField](https://docs.djangoproject.com/en/2.1/ref/contrib/postgres/fields/#daterangefield) which is an interesting alternative to doing the usual `start_date` and `end_date`.

If you're using Django Rest Framework, you are probably familiar with [django-filters](https://django-filter.readthedocs.io/en/master/). It's a generic system for filtering Django QuerySets, though I've mostly used it in combination with DRF.

## Timestamped

Tested with following versions:

- Django 1.11 (confident this will work with Django 2.1)
- django-filters 2.0.0 (won't work with previous versions, because 2.0 release includes a reworked version of RangeWidget)

## Requirements

Ranges are composed of lower and upper bounds that operate as a whole value. I want to be able to specify both bounds when filtering and get the objects that satisfy these exact values. By default, Django stores `DateRangeField` in its canonical form `[)` - lower inclusive and upper exclusive. In this case, I'm fine to specify and filter the bounds only in their canonical form. But it should be fairly easy to extend this to support different range types.

From the [filters list](https://django-filter.readthedocs.io/en/master/ref/filters.html#filters) I spot `DateRangeFilter` and `DateFromToRangeFilter`. `DateRangeFilter` defines a list of choices to select from (like today, yesterday past week, etc), not exactly what I'm looking for. `DateFromToRangeFilter` looks more appealing, as it allows filtering using the `_after` and `_before` suffixes. It will conveniently convert the passed strings to `DateField` and do `gte` and `lte` lookups on the field. Still not exactly what I need.

## Show me the code

{% highlight python %}
import django_filters
from psycopg2.extras import DateRange


class DateExactRangeWidget(django_filters.widgets.DateRangeWidget):
    """Date widget to help filter by *_start and *_end."""
    suffixes = ['start', 'end']


class DateExactRangeField(django_filters.fields.DateRangeField):
    widget = DateExactRangeWidget

    def compress(self, data_list):
        if data_list:
            start_date, stop_date = data_list
            return DateRange(start_date, stop_date)


class DateExactRangeFilter(django_filters.Filter):
    """
    Filter to be used for Postgres specific Django field - DateRangeField.
    https://docs.djangoproject.com/en/2.1/ref/contrib/postgres/fields/#daterangefield
    """
    field_class = DateExactRangeField
{% endhighlight %}

In your app:

{% highlight python %}
import django_filters

from .filters import DateExactRangeFilter
from .models import Event


class EventFilter(django_filters.rest_framework.FilterSet):
    event_dates = DateExactRangeFilter()

    class Meta:
        model = Event
        fields = [
            'event_dates',
        ]
{% endhighlight %}

## Explanation

Let's start from the bottom - `DateExactRangeFilter`. Since we are doing a regular lookup, we are inheriting directly from `Filter`. The `field_class` here is important, as it converts the raw values to a format and type we need, in this case `DateRange` composed of start and end dates. The `DateExactRangeWidget` lists the available suffixes.

Now you can do this `/api/events?event_dates_start=2018-08-07&event_dates_end=2018-08-12` to filter all events that start and end exactly on these dates.

## Warning

This solution is not ideal, because none of this documented. Django-filters doesn't provide a guide on how to extend existing classes to add your own filters, fields or widgets. The details of how these objects behave might change with time, hence use this at your own risk.
