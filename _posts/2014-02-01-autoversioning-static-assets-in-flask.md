---
layout: post
title:  "Autoversioning static assets in Flask"
date:   2014-02-01
---

Whether you are actively developing a web application or just occasionally making 
small changes, you will most probably face the problem with browsers caching static 
content you have (JavaScript, CSS, images). Basically it means that your users will 
see outdated version of your app and won't be aware of the brand new feature you 
were developing so passionately for the last week/month/year.

That's why versioning exists and you will see how easy it can be done in an automatic 
way.

![script version]({{ site.baseurl }}/assets/img/autoversioning/script_version.png)

### Requisites

The solution to caching problem can be implemented using any programming language, 
tool, framework, etc. Just to show you an example, I picked up [Flask](http://flask.pocoo.org/),
which comes bundled with [Jinja2](http://jinja.pocoo.org/docs/) templating language
for Python.


### How it works?

Let's say you have a CSS file and you included it in your HTML file. Now the trick 
is to give a different name to the CSS file when you are linking it to the template. 
But it must really tedious to change both the name of the stylesheet on hard disk and 
in the HTML. Therefore by using query string while linking in the HTML, we force 
the browser to reload the file and forget about the cache. At the same time there 
is no need to change the name of the file for real.

The version must be a unique number. Consider any of those:

 * file last modified timestamp
 * hash of the last commit
 * hash of the file 

### How to code it?

Most of the templating languages have filters. For example, in Flask and Jinja2 
it is very easy to create a custom filer. Suppose you have configured Jinja2 to load 
templates and it works fine. Now it's the time to create the custom filter.

{% highlight python %}
import os
from some_app import app


@app.template_filter('autoversion')
def autoversion_filter(filename):
  # determining fullpath might be project specific
  fullpath = os.path.join('some_app/', filename[1:])
  try:
      timestamp = str(os.path.getmtime(fullpath))
  except OSError:
      return filename
  newfilename = "{0}?v={1}".format(filename, timestamp)
  return newfilename
{% endhighlight %}

{% highlight html %}
{% raw %}
<script src="{{ url_for('static', filename='js/script.js')|autoversion }}"></script>
{% endraw %}
{% endhighlight %}

So we use a decorator to create a custom filter. The usage is exactly the same as 
for any other standard filter, by using a pipe. The first and only parameter that 
our function accepts is the path to the static file. 

Some manipulations are required in order to get the relative path to the file. 
For instance, we strip the first slash, otherwise `os.path.join()` will consider 
it an absolute path and will return this path. Then we need to get the timestamp 
of the last modification of the file, which is going to be a floating-point number that represents 
[the number of seconds since the epoch](http://docs.python.org/2/library/os.path.html#os.path.getmtime). 
If you want you can surround it in a `try-except` clause in case the file doesn't 
exist. The later is easy - just preppend the timestamp to the filename and return it. 

### Conclusion

A filter gave us an elegant and simple solution. Use the filter for any static 
files, whose contents will be updated sooner or later, but the name will stay 
the same.

If you want to see the filter implemented into a real project, then you can 
take a look at this [blog's github repo](https://github.com/ana-balica/codee_blog). 

### Update

Thanks to my friends, I have found out that query string to static assets is **not**
generally a good option. 

["Don't include a query string in the URL for static resources."](https://developers.google.com/speed/docs/best-practices/caching)
It says that most proxies will not cache static files with query parameters. 
Consequently that will increase the bandwidth, since all the resources will be 
downloaded on each request. 

"To enable proxy caching for these resources, remove 
query strings from references to static resources, and instead encode the parameters 
into the file names themselves." But this implies a slightly different implementation :)