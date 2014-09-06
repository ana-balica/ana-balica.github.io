---
layout: post
title:  "Custom Python Markdown Extension"
date:   2013-07-16
---

[Markdown](http://daringfireball.net/projects/markdown/) is awesome. It's a pleasure to use it and it's a pleasure to extend the Python Markdown module to fit your needs. Along with a bunch of official extensions, the module contains a nice [API](http://pythonhosted.org/Markdown/extensions/api.html) for writing your own extensions.

The API is clean and comprehensible. The reason of writing this article is that maybe somebody else will bump into the same problems like me and might find some of the solutions good enough to be used.

### Problem statement

Right now I am writing in Markdown. When I want to insert an image I am doing the following `![My Image](/static/img/my_image.png)`. I specify the relative path, because when I switch to different domain names, I want to have the same picture and no broken links. So I figured it out there can be 2 ways to solve that:

* render the template before markdown does. Use your templating language for inserting a variable that will be your `MEDIA_URL`.
* write a markdown extension that will fix your links

At this moment I would choose the first solution. But I implemented the second one, so I want to share some basic knowledge.
 
### My setup

I use [Flask-FlatPages](http://pythonhosted.org/Flask-FlatPages/) for rendering articles. Internally Python `markdown` module is used, and that means that you build your own extension easily.

### Consists of

The extension must have:

* a class that will represent the extension and needs to inherit from `markdown.extensions.Extension` and implement `extendMarkdown` method
* a function called `makeExtension()`, since you will be having per module one single extension. When you will specify the name of the extension (i.e. to the configuration variables of Flask FlatPages or directly when creating the Markdown instance), it will import the module and call that function. Here I am just quoting the documentation.
* some kind of processor that will perform the manipulations with your markdown text

### Let's start coding

{% highlight python %}
def makeExtension(configs=[]):
    """ Return an instance of the AbsoluteImagesExtension """
    return AbsoluteImagesExtension(configs=configs)
{% endhighlight %}

This function also accepts a list of configs. It is useful for us, since we want to get a base URL or a media URL which we will prepend to the image source links.

There are different flavors of processors: **pre**processor (before code is sent to Markdown core), **tree**processor (being an [ElementTree](http://effbot.org/zone/element-index.htm) object), **post**processor (you got your output string already). For my needs I have decided that an ElementTree is a good way to easily extract `<img>` tags and insert the changed `src`. 

{% highlight python %}
class AbsoluteImagesTreeprocessor(Treeprocessor):
    """ Absolute Images Treeprocessor """
    def run(self, root):
        imgs = root.getiterator("img")
        for image in imgs:
            if self.is_relative(image.attrib["src"]):
                image.set("src", self.make_external(image.attrib["src"]))

    def make_external(self, path):
        return urljoin(self.config["base_url"], path)

    def is_relative(self, link):
        if link.startswith('http://'):
            return False
        return True
{% endhighlight %}

Method `run()` is mandatory. The syntax is straight forward and the implementation is pretty naive. Also pay attention that we have the access to the config.

And finally the extension class:

{% highlight python %}
class AbsoluteImagesExtension(Extension):
    """ Absolute Images Extension """

    def __init__(self, configs=[]):
        self.config = {
            'base_url': [None,
                         "The base URL to which the relative paths will be appended"],
        }

        for key, value in configs:
            self.setConfig(key, value)

    def extendMarkdown(self, md, md_globals):
        absolute_images = AbsoluteImagesTreeprocessor(md)
        absolute_images.config = self.getConfigs()
        md.treeprocessors.add("absoluteimages", absolute_images, "_end")

        md.registerExtension(self)
{% endhighlight %}>

`self.config` should be a dictionary and store values of the form `param_name: [param_value, description]`. In the `extendMarkdown()` we are adding our treeprocessor class to Markdown TreeProcessors by specifying it's name, an instance and where to insert it - before the end.

The extension lives here - [https://gist.github.com/ana-balica/5944798](https://gist.github.com/ana-balica/5944798).


### Enabling the extension

For Flask do something like that - `FLATPAGES_MARKDOWN_EXTENSIONS = ['absolute_images(base_url=http://this-important-url)']`  in the config. First Markdown will search for the extension in the module directory - `markdown.extensions`. If you don't want to keep it there for some reasons, you can make the extension name contain dots (then Markdown will import it as-is) or prepend to the extension filename `mdx-` (`mdx-absolute_images`). But be careful that the `mdx-` notation is discouraged from being used.
