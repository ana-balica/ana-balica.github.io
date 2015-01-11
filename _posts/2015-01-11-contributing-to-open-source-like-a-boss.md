---
layout: post
title: "Contributing to open source like a boss"
date: 2015-01-11
tags:
  - code
  - open source
---

This is a collection of the most obvious, nevertheless overlooked, bullet points
that every contributor should be aware of when making the first contribution
to an open source project.

I remember my first contribution to
[MoinMoin Wiki Engine](http://moinmo.in/) and it was terrible. And I'm not talking about code
(which was probably also pretty bad), but about all the surrounding elements
to an open source contribution. With time you learn how to do it the right the
way.

Now I find myself in the position of the reviewer and I see tons of
mistakes.

<img src="{{ site.baseurl }}/assets/img/contributing/noobs_everywhere.jpg"
alt="Noobs everywhere" style="width: initial;" class="center">

But it's OK. I was a noob quite recently and for some projects I am still holding
on to my noob award. Everybody passed through the stage of first contribution.
But once you learn and get comfortable, it's very false to start assuming that
newcomers should catch on very fast. I think newcomers need help and guidance.
That's why I figured it's a good idea to disseminate some of the common rules
of open source world.

### How do I get started?

If you made the decision to contribute, don't go to the organization's
communication channel and say that out loud. That's not bad, it's just useless.
Long term contributors and maintainers are like smart girls dating teenage boys:
they ignore everything boys say and only pay attention to what they do.

And since you set your mind to contribute, take some time to search for project
documentation and read it. The question *How do I get started?* is mostly
irrelevant. I find it kind when people on the channel point out to relevant links
and give short instructions. But... come on! Open source projects' documentation
is not a hidden treasure chest, it's out there indexed by Google. Don't be lazy.

<img src="http://imgs.xkcd.com/comics/rtfm.png" title="Life is too short for man pages,
but occasionally much too short without them." alt="RTFM" style="width: initial;" class="center">

Spend at least an hour trying to find relevant developer documentation. It's
perfectly OK to ask for help if the docs are missing, lacking or misleading. In
fact, if you figured the documentation is not straight-forward, express your
concern and propose a solution. Your feedback will be appreciated if the
community is welcoming.

### Follow that guide

Usually a community that cares about code quality will comply to a code style guide.
This is a very important document, read it carefully. It will teach you about
naming conventions, do's and don'ts and other recommendations. For example, in
Python world a lot of projects follow [PEP8](https://www.python.org/dev/peps/pep-0008) -
if you haven't seen a style guide, here is what it looks like.

Pay attention to the tools the community uses. They might prefer one pastebin
over another, have a community-wide wiki, expect people to use a single code
review tool. Follow those recommendations. It makes everybody's life easier.

### Find easy TODO

Projects usually have a bug tracker, whether it's a Trac tracker, GitHub issues
or simple wiki page. Doesn't matter as long as the tracker is up-to-date. Try to
see if they have tickets with *easy* tag or something similar. Search for the one you feel most
confident about.

If you found a bug while using the software, check if it's being already listed.
If not, report the problem.

Further express your willingness to work on the ticket: assign it to yourself or
leave a comment - whatever works best. The reason is nobody wants duplicate
work.

Well, now is your time to shine.

### Branch that out

Most probably you will need to create a new feature branch. Holds true for git,
somewhat for mercurial. Name the branch appropriately. Don't name
it *stephen* or *my_branch*.

Check what is the leading branch for development, might be *master* or maybe *dev*.
Create the feature branch from the leading development branch. Pull the latests
changes. If the branch was updated while you were working on your feature branch,
rebase your work. It's so much easier to merge the contributions of others if
their branches contain the latest changes.

Do this for every feature. It might seem like overhead, but it actually saves
some pain in the future.

### Run those tests

After you solved the ticket, take some time to run all the necessary scripts
before committing. Those might be unittests, UI tests, linters. Pay attention
to their output. If the output makes little sense to you, take a step back and
read about the tool to figure out the meaning of the output.

It's pretty cool when a project uses a CI (Continuous Integration) tool. It tells
the developers straight away if something is wrong. Thought that doesn't spare
you and others from submitting clean working code.

Seriously, run those tests.

### Code review

Go back to the docs and search for the section that tells you where and how to
submit code for review.

There are plenty of code review tools: [Gerrit](https://code.google.com/p/gerrit/) ,
[Rietveld](https://code.google.com/p/rietveld/), [Phabricator](http://phabricator.org/),
[GitHub pull requests](https://help.github.com/articles/using-pull-requests/).
So there must be one that community prefers. Before you
put your code on review, check if you are submitting only the relevant pieces,
remove all the debug code, don't submit sensible information (passwords, keys)
or local user configuration.

Your code review should contain a title and a description if necessary. Don't
name it *First contribution* or *My first pull request please review asap*.

And once you did submit the code, go to that link and **review it yourself**. It's
hard to emphasize how important is that step. Take a minute or two to scan with
your eyes all the code. You will be surprised to find out that you have slipped
some unwanted code in there or you just found a better solution.

### Pull request

Pull request process can be merged with the code review process. Contributors can
review your code once you make the pull request. Or they might be separate.
Matter of preference.

Repeating myself, but clean title, description, commit messages are important.
Refer to the ticket in commit message or pull request title if it's necessary.

Limit a pull request to one to 3 commits. Don't overload code reviews and pull
requests with tons of code. It's almost impossible to review big chunks of code.
A method, a function, a small class, some bugfixing - OK. Class with 10 methods,
extra 5 templates with controllers and routes - not OK.

**Reviewing isn't easy**. It takes quite some time and the reviewer holds same
responsibility for your code as you do, since he or she  decides to accept or
discard your contribution.

Keep it short. Keep it simple. And people will love it.

### Wrap-up

It's OK to make mistakes, as long as we acknowledge and fix them. The rules we
have imposed ourselves help keep the codebase readable and maintainable, hence
ensure the longevity and popularity of the project. Comply to this set of rules
and, boy, you're gonna become a pleasant person
to work with.

Open source is a good place to learn from others. Don't miss that opportunity.
Appreciate the feedback.

