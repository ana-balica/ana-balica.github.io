---
layout: post
title: "Intricacies and optimization of Datastore indexes"
date:   2016-11-28
tags:
  - app engine
  - datastore
---

It's nice that development server is so kind to generate Datastore indexes for us, unfortunately those aren't always optimal and might require some tweaking from our side. Hence it's important to be knowledgeable on intricacies of index creation.

This is a follow up post to [Datastore indexes basics]({{ site.baseurl }}/2016/11/20/datastore-indexes-basics/) that covers essentials, mentions the dependencies used and contains some base models that we will be using here.

## Speed considerations

The reason why we want to optimize indexes is speed. Simply put the more simple and composite indexes an entity has, the slower it becomes to do writes. That's why Datastore imposes limits on the number and size of index entries ([200 composite indexes and 2 Mib maximum sum of the size of an entity's composite index entries](https://cloud.google.com/datastore/docs/concepts/limits)). It is possible by having a large number of properties on an entity/model or having a huge amount of composite indexes in `index.yaml` to exceed those limits though.

## When order matters

Order in the **WHERE** clause *doesn't* matter. If you have multiple equality filters on different properties, then those 2 indexes are equivalent and one of them can be safely discarded. Going back to our example with a Book model. If we execute this query:

{% highlight python %}
Book.objects.filter(author='Mark Twain', title='Some title').order_by('-published_date')
{% endhighlight %}

Development server will create this index:

{% highlight yaml %}
- kind: library_book
  properties:
  - name: author
  - name: title
  - name: published_date
    direction: desc
{% endhighlight %}

It can serve this query as well (we have swapped in place author and title properties):

{% highlight python %}
Book.objects.filter(title='Some title', author='Mark Twain').order_by('-published_date')
{% endhighlight %}

It is possible to confuse automatic index creation and make it also create this index:

{% highlight yaml %}
- kind: library_book
  properties:
  - name: title
  - name: author
  - name: published_date
    direction: desc
{% endhighlight %}

Then it's our duty to spot indexes alike and remove duplicates.

In general Datastore will try to create and insert indexes in alphabetical order with properties inside of an index ordered like that:

* Properties used in equality filters
* Property used in an inequality filter (there can be only one)
* Properties used in sort order

That said, the **direction** of a property *makes* a difference, hence `order_by('published_date')` is not the same as `order_by('-published_date')` and 2 indexes should be created respectively.


## Minimum vs perfect

The index that we created above is a perfect index - the whole query is served by a single index. This allows the most efficient and fast querying. In contrast we could split the perfect index into 2 minimum indexes:

{% highlight yaml %}
- kind: library_book
  properties:
  - name: author
  - name: published_date
    direction: desc

- kind: library_book
  properties:
  - name: title
  - name: published_date
    direction: desc
{% endhighlight %}

These 2 indexes now serve 3 different complex queries:

{% highlight python %}
Book.objects.filter(author='Mark Twain').order_by('-published_date')
Book.objects.filter(title='Some title').order_by('-published_date')
Book.objects.filter(author='Mark Twain', title='Some title').order_by('-published_date')
{% endhighlight %}

To serve the last query, Datastore uses [zigzag merge join algorithm](http://www.youtube.com/watch?v=ofhEyDBpngM). To be able to merge those 2 indexes to obtain results for the last query, both indexes have to be ordered the same way.

It might not seem like a major winning strategy. In our case we have reduced 3 indexes to 2 - not a big deal. But consider a different example where we add more properties to the Book model and we create an interface where librarians can choose by which properties to filter and to sort. If we try to predict every single combination and create perfect indexes for each, we will obtain fast querying, nevertheless we might easily reach the index cap and the operation of creating or updating books will become slower.

Consider we have 6 properties for filtering and we allow ordering by 4 properties. That'll result in 256 perfect indexes (whoops, we've reached the index cap). It's computed the following way:

{% highlight text %}
2^(number of filters) * (number of different orders)
2^6 * 4 = 256
{% endhighlight %}

In this case we should resort to creating minimum indexes or a combination of minimum and perfect indexes. The absolute minimum to serve all the possible queries in this case is 24 indexes (`6 * 4`). This might not be the optimal solution though, some queries might be executed more often than others, so maybe it makes sense to add another 3-4 perfect indexes to serve most common use cases.

Besides when resorting to minimum indexes, we should be aware of the shape of the data. Merging using the zigzag merge join algorithm comes with a cost. Poor performance is achieved when many entities match each scan in separation, but few entities match the query as a whole. App Engine has an excellent explanation on that in [Datastore Index Selection & Advanced Search](https://cloud.google.com/appengine/articles/indexselection) article.

## Exploding indexes

Iterable fields complicate things. Normally each row in the index table represents one entity in Datastore. Take this index `Index(Book, author, title, -published_date)`. If we have 10 books with those properties set, then Datastore will create 10 rows in the index table.

If we add another indexable property to Book `keywords = ListField(CharField)`, then the number of rows becomes the number of unique values in the ListProperty. Datastore will create an entry for each combination of property values.

Now if we use multiple multi-valued properties, then the number of entries for each entity can "explode" combinatorially. Even though it's a single index, such an index can dramatically increase the cost of writing an entity to Datastore.

Let's amend our Book model:

{% highlight python %}
class Book(models.Model):
    title = models.CharField(max_length=200)
    ...
    keywords = SetField(CharField)
    ratings = ListField(DecimalField)
{% endhighlight %}

And create another index:

{% highlight yaml %}
- kind: library_book
  properties:
  - name: title
  - name: keywords
  - name: ratings
{% endhighlight %}

With the following new instance:

{% highlight python %}
book = Book.objects.create(
    title="My sci-fi book",
    keywords=['sci-fi', 'dystopia', 'post-modern'],
    ratings=[4.3, 4.9, 3.9, 4.1, 4.6]
)
{% endhighlight %}


Datastore will create 15 entries (`|keywords| * |ratings| * |title|`) for this index. The larger the number of items in an iterable property, the more rows will be created in the index table. This is what is called *exploding indexes*.

To ameliorate such a hungry index, we can split it up into 2 indexes (same zigzag merge join algorithm applied in case of a query that filters and sorts on those 3 properties):

{% highlight yaml %}
- kind: library_book
  properties:
  - name: title
  - name: keywords

- kind: library_book
  properties:
  - name: title
  - name: ratings
{% endhighlight %}

For this example Datastore will create 8 entries (`|keywords| * |title| + |ratings| * |title|`) in contrast with 15 entries in the index table. Don't forget to vacuum clean indexes, once you update them.

Unexploding indexes might not always be a necessity, since (again) it all depends on the shape of your data. You might be fine with a single index that contains 2 iterable properties if one of those is a property that always has only one value. Maybe it was created with a thought about distant future, where more options can be added without the need to migrate all the data.

## Dead indexes

Large projects with a lot of models might suffer from dead-index weight. Adding indexes is easy, removing them - not so much. You never know if there's a hidden query somewhere that your search through the codebase didn't spot. So maybe we should just keep that index? Nope.

There are different scenarios when we can and should identify dead indexes:

1. Removing a query. That query might have required an index in the past. We can delete it, but we also need to check the rest of the code if nobody else is using this index. It can be also be a minimum index used in a mix with other indexes to serve a complex query.
2. Changing Django Meta class ordering. This kind of change will have repercussions all over the code and most probably all the indexes will have to be updated. Big changes ahead!
3. Scanning for duplicate indexes, whether it's obvious duplicates with properties in different orders or hidden duplicates with equivalent indexes just in a different form (think minimum vs perfect).

Either way we need to practice mindfulness to keep our indexes optimal and at a minimum.
