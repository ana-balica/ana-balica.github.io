---
layout: post
title:  "OMG! git reset --hard"
date:   2013-08-28
tags:
  - code
  - git
---

I bet you know that jaw-dropping command `git reset --hard <commit-hash>`. Remember that first time you lost all that code? No? Well, good for you you didn't.

Fair enough the command is considered potentially dangerous. I have even seen blogposts about git reseting that make it hard to detect the command in-need (even if the user came for a full delete/erase/destroy this commit and the data it contains). The authors add lots of explanations around, so you need to read first and then type the **correct** command. No copy-paste, no tl;dr allowed.

In fact as I learned from [Pro Git](http://git-scm.com/book) the dreadful harm is reversible...

## Lost my commits

So let's take an example. Suppose you just have started a new project and your history looks like this:

{% highlight bash %}
$ git ls
f1c16f4 (HEAD, master) second [ana-balica]
743c90d first [ana-balica]
{% endhighlight %}

\* **ls** is an alias for some pretty formatting

Now you type `git reset --hard 743c90d`, which resets your HEAD at `743c90d` commit. But actually that last commit is not lost. For a recovery all you need to know is the SHA of that lost commit. Create a new branch starting with this commit, rebase or merge your master branch with the new branch that contains the commit and you are saved.

{% highlight bash %}
$ git branch recover-branch f1c16f4
$ git rebase recover-branch
{% endhighlight %}


It's good if you have the SHA of the lost commit(s). If not, try those steps in the following order:

* check the terminal output, at some point you have typed `git log` and maybe you have the hash
* try `git reflog` - it records your branch tip changes. Look for the commit name if you remember it. Mine looks like that:

{% highlight bash %}
$ git reflog
743c90d HEAD@{4}: reset: moving to 743c90d
f1c16f4 HEAD@{5}: commit: second
743c90d HEAD@{6}: commit (initial): first
{% endhighlight %}

* use `git log -g` if you still can't comprehend which is the one

## Lost stagging area

Worse then reseting to a commit in the past, it is reseting the HEAD - `git reset --hard HEAD`. But don't worry if those were in your staging area, which means your index knew about them. Type `git fsck --lost-found`. It checks the validity of objects in the database and therefore you will see your dangling blob there. You can open it and check it's contents. For recovering all that data simply dump it's contents to the file `git show 898910b > test.txt`, where the SHA is the blob's SHA.

## Lost unstagged area

The horror is here! At this point you have made some changes to your files, never `git add`ed them, and then the magic happen and wiped away all that. Must be a crisis in your life, you are doomed, life doesn't make sense any more...

![The Scream]({{ site.baseurl }}/assets/img/git-reset/scream.jpg)

Maybe you are lucky and you can find the light at the end of tunnel. Check that:

* maybe Ctrl+Z?
* see if your smart IDE has any internal version control system, personally I use PyCharm and it is very easy to view and recover from history. Protect yourself, use an IDE ;)
* are the changes recorded anywhere else? For instance if your project is located in Dropbox (I know that's stupid if your have version control, but who knows), then you can see previous versions on Dropbox website and restore to any version you want. A new version is recorded on a new save. Don't leave it for tomorrow, because only the last 30-days versions are stored in the cloud.


If you have some other ideas on code recovery, feel free to share :)
