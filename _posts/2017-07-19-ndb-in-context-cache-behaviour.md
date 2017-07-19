---
layout: post
title: "NDB in-context cache behaviour"
date:   2017-07-19
tags:
  - app engine
  - datastore
---

NDB allows an App Engine app to connect to Cloud Datastore.
It helpfully provides 2 types of caching enabled by default: memcache and
[in-context cache](https://cloud.google.com/appengine/docs/standard/python/ndb/cache#incontext)
(also referred to as in-memory cache).

## Timestamped

App Engine Python SDK 1.9.57

## Anything wrong with it?

Yes and no. See this:

{% highlight python %}
>>> box = LegoBox.query().get()
>>> box.pieces_count
32
>>> box.pieces_count = 64  # set the value, don't save
>>> LegoBox.query().get().pieces_count
64
>>> LegoBox.query().get(use_cache=False).pieces_count
32
{% endhighlight %}

When fetching a fresh object even within the same thread,
one would expect to get the value from the database and not the one from an in-memory cache.
This is considered [working as intended](https://issuetracker.google.com/issues/35897973).

A happy clueless developer could write this:

{% highlight python %}
def test_smth():
    box = LegoBox(box_type='architecture', version='v1')
    box.put()
    box.count_pieces()

    # Refresh the box object and check the result
    box = box.key.get()
    assert box.pieces_count == 544
{% endhighlight %}

`LegoBox.count_pieces` has a bug in it where it forgets to call `put` (bugs happen y'know).
Unless one is diligent to always disable the in-context cache for the fetches,
this test isn't good enough,
because it doesn't expose the bug.
YOLO!

Sadly this behaviour is not mentioned in the official documentation,
so there's a high chance one will discover this "desired" effect in a bitter situation while debugging for hours.

## How to disable in-context cache

First strategy is to pass `use_cache=False` when making queries.

Second option is to disable the cache per model. In theory this will slow down your app.
In practice, I don't have benchmarks, so I don't know.
Moreover it always depends on what you are doing.

{% highlight python %}
class LegoBox(ndb.Model):
    _use_cache = False
{% endhighlight %}

Third stategy is to set a global policy to turn off automatic caching for all NDB models and be done with it.

{% highlight python %}
# appengine_config.py
from google.appengine.ext import ndb

context = ndb.get_context()
context.set_cache_policy(lambda key: False)
{% endhighlight %}

---

I was a happy-go-lucky developer who wrote lots of passing tests that interact with NDB models,
only later to discover that the code is failing in production.
Luckily people smarter than me pointed out what might be the cause.
This is a cautionary story to help one avoid such issues.
