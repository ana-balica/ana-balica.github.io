--- 
layout: post 
title: "Systers Portal"
date:   2014-11-11 
---

Today I was listening to [Swift](http://www.programmingthrowdown.com/2014/09/episode-36-swift.html) 
episode by *Programming Throwdown* podcast
series. At some point hosts raise the question of open source involvement. You probably
also received this piece of advice:

> If you want to get your hands on some programming as a beginner or a recent
> graduate, try doing some open source.

That is rather vague. 
What most people skip saying is what kind of open source work should you start with. Hosts
of the show point out the fact that it's not necessary to jump right into big 
popular projects, like Linux Kernel or PostgreSQL, simply because it will take tons
of time to read and understand the existing codebase and get your patch accepted by
the core maintainers. So the solution is to seek for a small open source 
project, that involves a technology you like or want to get better at it, preferably
with few developers involved.

Systers Portal is one of those one-man-show projects, that might suit very well 
the open source beginners. 

### What is that?

[Systers](http://systers.org/) is the world's largest email community of women 
in technical roles in computing. It is indeed huge and covers a lot of technical
areas. It started as a small mailing list and grew to 4,000 
members from all over the world. Considering the modern needs, a single mailing list
isn't very nifty to manage such a huge community.

Hence the idea of a portal came up. Systers Portal is a unified platform for 
Systers and its sub-groups to share information and get the latest news. This is
a place to manage sub-communities, latest news, interesting and useful 
resources, community members.

Systers Portal was started in summer 2014 as a 
[Google Summer of Code](http://www.google-melange.com/gsoc/homepage/google/gsoc2014) 
project involving two students: me and Chitra Khatwani, under the mentorship of Lynn Root, 
Rosario Robinson, Promita Bose and Laura Cassell. We launched a prototype, 
hosted on [portal.systers.org](http://portal.systers.org/).

*Disclaimer: the prototype hosted on [portal.systers.org](http://portal.systers.org/) 
isn't going to match exactly your development version from the official repository. The 
development version incrementally adds features present in the prototype.*

### Reasons

Here are some of the obvious reasons why Systers Portal is so good for you:

* It is a Django project, build using best practices and techniques, with 
  readability and maintainability in mind. We try to keep up with the latest 
  releases, hence we are using Django 1.7 with a set of up-to-date packages.
* We are strict about styleguides and code readability. You will learn fast
  about one of my favorite Python mantras, called [PEP8](https://www.python.org/dev/peps/pep-0008).
* We have [100% code coverage](https://coveralls.io/r/systers/portal?branch=master)
  and care about unittests.
* We are hosted on [GitHub](https://github.com/systers/portal), ready to be 
  forked and accepting pull requests.
* We have [documentation](http://systers-portal.readthedocs.org/en/latest/). 
  Particularly documentation how to get started, what should you know before 
  getting down to code, how to run the project locally. The 
  [First Contribution](http://systers-portal.readthedocs.org/en/latest/develop/contributing.html)
  article will drive you through all the necessary steps to make your first
  contribution to Systers Portal. The code contains helpful docstrings as well.
* There is currently one developer working on Portal - it's me. It is generally
  easier to read a single mind expressed in code, than a whole bunch. This
  doesn't imply that the code is always obvious and clear. Particularly 
  every time I review old pieces of code, I rewrite something. I need your
  help to see the errors and solutions.
* We have a list of [easy TODOs](http://systers.org/systers-dev/doku.php/gci14systers:portal) 
  (initially created for Google Code-in).
* By contributing to Systers Portal, you will help women from technical fields
  to enjoy a friendly environment via a unified portal.

### Links! Links! Links!

* GitHub repo - [github.com/systers/portal](https://github.com/systers/portal)
* Documentation - [systers-portal.readthedocs.org](http://systers-portal.readthedocs.org/)
* Systers Portal prototype - [portal.systers.org](http://portal.systers.org/)
* About Systers - [systers.org](http://systers.org/)
