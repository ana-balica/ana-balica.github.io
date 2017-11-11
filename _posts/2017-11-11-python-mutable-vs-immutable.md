---
layout: post
title: "Python: mutable vs immutable"
date:   2017-11-11
tags:
  - python
---

Mutable means an object can be modified after it's been created.
In contrast immutable is an object whose state can't be changed after its creation.
Python has both mutable and immutable data types.
There's a simple recipe to figure out if an object can be "mutated":

* initialise a variable of the target type
* store the ID
  (Python has a build-in `id()` function. In CPython this is the address of the object in memory.)
* mutate the variable
* check the previously stored ID vs the current ID of the variable

## Mutable types

Take lists.

{% highlight python %}
>>> salad = ['spinach', 'tomatoes']
>>> id_before = id(salad)
>>> salad += ['eggs']
>>> id_after = id(salad)
>>>
>>> id_before == id_after
True
{% endhighlight %}

I created my original salad with 2 ingredients. Then I added another ingredient - eggs.
Even after the mutation (adding eggs), the salad object is still pointing to the same place in memory,
it has the same ID.

Even though we have used the `+` to mutate the list, Python exposes many methods
that imply that lists are mutable: `list.append`, `list.pop`, `list.remove`.
With its API design programming languages usually try to tell us what is mutable and what is not.

## Immutable types

Take strings.

{% highlight python %}
>>> my_string = 'foo'
>>> id_before = id(my_string)
>>> my_string += 'bar'
>>> id_after = id(my_string)
>>>
>>> id_before == id_after
False
{% endhighlight %}


Here we are doing something similar to what we did with the `salad` list - use `+` to append to the original string.
However in this case the object after the mutation happens to reside in a different place in memory.

In fact, Python doesn't really provide any methods to modify a string "in place" (or give us the illusion we can do that),
hinting on the fact that strings are immutable.

----

Now you're able to understand better how mutability works.
Have fun figuring out the mutability of the other Python data types!
