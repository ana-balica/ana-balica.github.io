---
layout: post
title: "xkcd getter"
date:   2013-04-25
tags:
  - code
  - python
---

Some time ago I had an assignment to implement a crawler. Now I understand that I didn't do the right thing, since a crawler is a program that will follow links in a systematical manner on every page. What I did is called a spider - a program that downloads web pages. Well, you can take your spider, add some parsing, extract the necessary links, follow them, download web pages and basically this is a crawler.

Nonetheless I still have that spider written in Python. It suffered many changes, since initially it was implemented using sockets, so far now it uses the `urllib`. Since I enjoy reading [xkcd](http://xkcd.com/), this website became my target.

The idea to create this post came out of this [comic](http://xkcd.com/859/):

<div style="text-align: center;">
  <img src="http://imgs.xkcd.com/comics/(.png" alt="xkcd comic n# 859" style="width: 300px;">
</div>

Tried my spider on it and it didn't crash, gave me the title, the caption and the picture - yey!

### Text or screencast

If you are a visual person, then you can watch the screencast below. It describes every single line of code written. Important: the version of the script used in the screencast is a bit outdated, some design solutions were changed, but if you are interested mainly in parsing, then the video is eligible to be watched.

<iframe width="100%" height="315" src="http://www.youtube.com/embed/BmLDo8mR4Nk?list=PL_6_YrF4ZAJ-Sj34pZUmFeTl9AiBrJz6f" frameborder="0" allowfullscreen></iframe>

If you don't like watching looong screencasts (like me), then here is a short overview of the script. Also you can find the source code [right here](https://gist.github.com/ana-balica/5454270).

### Prerequisites

You will need:

1. Python 2.7
2. [lxml library](http://lxml.de/)
3. optionally a browser extension to check you xpaths. I am using [XPath Helper](https://chrome.google.com/webstore/detail/xpath-helper/hgimnogjllphhhkhlmebbmlgjoejdpjl) for Chrome

### Downloader

Create a separate class that will hold the download functionality. Here is the snippet:

{% highlight python %}
class Downloader():
  '''
  Class to retreive HTML code
  and binary files from a
  specific website
  '''

  def __init__(self, url):
    self.url = url

  def download(self, image_name='', is_image=False):
    try:
      browser = urllib.urlopen(self.url)
      response = browser.getcode()
    except:
      print 'Bad connection'
      sys.exit()

    if response == 200:
      contents = browser.read()
    else:
      print 'Bad header response'
      sys.exit()

    if is_image:
      self.save_image(contents, image_name)

    return contents

  def save_image(self, contents, image_name):
    image_file = open(image_name, 'wb')
    image_file.write(contents)
    image_file.close()
{% endhighlight %}

Pass to the constructor the URL where from to download the contents. There are 2 methods, from which only the `download()` represents the interface of the class. `urllib.urlopen()` method opens a socket for you establishing a connection. It is a good idea to wrap this part of code into a `try/except` since it can raise an exception if the connection cannot be made (many reasons for that). Afterwards check if the server responded with `200 OK` and in case of success get the contents. Those contents is the HTML code of the page.

There is also a scenario when the downloaded content is an image (binary file). For that we need explicitly to pass the `is_image=True` parameter. An image will be saved on the local machine in the current directory (where the script is placed).


### Parser

Parser class is a not a generic class. It is build specifically for parsing [xkcd.com](http://xkcd.com/). If you plan on extending the spider, then it might be a good idea to create an interface `Parser` and let other classes implement that interface.

Since the `xkcdParser` class is a bit long, we will examine it in chunks.

{% highlight python %}
class xkcdParser():
  '''
  Class for parsing xkcd.com
  '''

  def __init__(self):
    self.url = "http://xkcd.com/"
    self.last_comic_nr = None
    self.contents = ''
    self.title = ''
    self.caption = ''

  def set_last_comic_nr(self):
    downloader = Downloader(self.url)
    self.contents = downloader.download()
    self.last_comic_nr = re.search(r"http://xkcd.com/(\d+)", self.contents).group(1)
    self.last_comic_nr = int(self.last_comic_nr)

  def get_title(self):
    if self.contents:
      tree = etree.HTML(self.contents)
      self.title = tree.xpath("string(//div[@id='ctitle'])")

  def get_caption(self):
    if self.contents:
      tree = etree.HTML(self.contents)
      self.caption = tree.xpath("string(//div[@id='comic']/img/@title)")

  def get_comic(self):
    if self.contents:
      tree = etree.HTML(self.contents)
      url = tree.xpath("string(//div[@id='comic']/img/@src)")

      downloader = Downloader(url)
      downloader.download(self.title, True)
{% endhighlight %}

Keep the URL of the website in a class attribute. Besides the constructor there are a bunch of methods here.

`set_last_comic_nr` is... well self-explanatory. Why do we need it? Because we plan on allowing the user to specify the number of the comic and also generate a random number in a specific range. Not the best solution to create a `Downloader` object inside the method, but it works for this short spider.

After you get the contents, you need to parse them. Two solutions (might be more, but the most mainstream) are xpaths or regular expressions. If you go to the home page of [xkcd.com](http://xkcd.com) then below the image you will see a permanent link for this comic and it has the `id` we are looking for - the id of the most recent comic. Using the inspector we can check that this part of HTML is not wrapped into a tag, the text is placed between `<br>`s after an `ul`. That's definitely not a well-formed XML mark-up.

Even in this case it is possible to get the data with the following xpath - `//div[@id='middleContainer']/text()[preceding-sibling::br][1]`. That will return the following string - 'Permanent link to this comic: http://xkcd.com/1203/', which again should be parsed with regular expressions.

I've taken a shortcut. My regexp searches for one or more digits after the string `http://xkcd.com/` through the whole DOM and groups those digits. Regular expressions are slow, in my case it was necessary to build a whole automata out of the contents, therefore you need to avoid them as often as possible.

The other 3 methods are already using xpaths to search for the title, caption and path to the image through the contents of the page. The method `get_comic()` also instantiates a `Downloader` object and downloads the image.

The xpaths are fairly simple, but anyway let's examine of them. How about the `string(//div[@id='comic']/img/@src)`? English translation is the following -

* search through the DOM for a div with an `id` that equals 'comic'
* inside that node search for an `img` tag
* get the `src` attribute of that `img` tag
* strigify everything you have found - if you don't use the `string` function, then xpath will return an array with every matched element.


The rest of the code looks like that:

{% highlight python %}
class xkcdParser():

  def get_current_comic(self):
    self.set_last_comic_nr()
    self.get_title()
    self.get_caption()
    self.get_comic()

  def get_comic_by_id(self, comic_nr):
    if not self.last_comic_nr:
      self.set_last_comic_nr()

    try:
      comic_nr = int(comic_nr)
    except:
      print 'The comic number should be an integer'
      sys.exit()

    if comic_nr <= self.last_comic_nr:
      url = self.url + str(comic_nr)
      downloader = Downloader(url)
      self.contents = downloader.download()
      self.get_title()
      self.get_caption()
      self.get_comic()

  def get_random_comic(self):
    if not self.last_comic_nr:
      self.set_last_comic_nr()

    comic_nr = random.randint(1, self.last_comic_nr)
    self.get_comic_by_id(comic_nr)
{% endhighlight %}

We need 3 more methods: get the latest comic, get a specific comic and get a random comic. `get_current_comic()` actually has a problem, because it hides the loading of contents. Since we call that method that will set last comic number, we know that it will also set the contents ad there is no need to download the page again. Still, prefer a different implementation for that.

Before downloading a comic with an id, first check the data that you get from the user - if it's an integer and if it's in range, perform the download, setting of title, caption etc. The `get_random_comic()` uses the method above, initially generating a random number.

Aaaand we are done!

You can get the entire snippet by accessing this gist - [https://gist.github.com/ana-balica/5454270](https://gist.github.com/ana-balica/5454270)


