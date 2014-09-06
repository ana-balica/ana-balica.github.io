---
layout: post
title:  "CLI and GUI git"
date:   2013-04-21
---

To be clear from the very beginning, I vote for CLI git to start with. There's nothing better than understanding what exactly you are doing and not just try and click every available button. Just imagine yourself connecting to a remote server that doesn't have your favorite GUI client.

During my interaction with git (which started not so long ago), I've developed a hybrid workflow. There are lots of git clients and to be honest, I haven't tried many of them, so I'll just present the one I'm familiar with - **gitg**.

### History

Visualizing your tree in git is definitely a very useful thing. To achieve the following you can use the `--graph` argument. If you leave the default `log` command then the long commit messages will make it hard to read the topology.

Use pretty formatting and create an alias for that:

{% highlight bash %}
[alias]
ls = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate
{% endhighlight %}

Therefore the command `git ls --graph` will output the following:
{% highlight bash %}
\*   46be671\ Merge branch 'development' of github.com:author/repo into development\ [author]
|\  
| * fe8bef7\ commit nr 5\ [author]
| *   b801dc6\ Merge branch 'development' of github.com:author/repo into development\ [author]
| |\  
| | * ace6bdb\ commit nr 4\ [author]
| * | 2fd916a\ commit nr 3\ [author]
| * | 7419237\ commit nr 2\ [author]
* | | fd0c4bf\ commit nr 1\ [author]
{% endhighlight %}


Not bad, but a GUI client makes it better. Basically it's the same branch topology , but it offers an intuitive way to crawl though commits, see the details, files that were changed, the branch pointers.
![gitg - history tree]({{ site.baseurl }}/assets/img/git/gitg1.png)


### Diffs

`git diff` shows changes between working tree, commits, branches, etc. It can take lots of arguments and is very flexible. In case you just don't want to remember all those possibilities, the client will provide snippets with diffs for lots of scenarios. It becomes handy when you want to review what you are about to commit or what what do you want to add to your staged area, what is inside your stashes, what have you committed long ago.

![gitg - see the diffs]({{ site.baseurl }}/assets/img/git/gitg2.png)


### Stash

I use stashing all the time. And I also try not to keep them there for a long time. So mainly I do something like this:

{% highlight bash %}
$ git stash
$ do some stuff here
$ git stash pop
{% endhighlight %}

When you `pop` a stash, this means that you apply your stash to the current working tree and also drop it from the stash list. That's a handy shortcut.

If you want to check the diffs inside your stash type `git stash show -p stash@{0}`. Not the most pretty stash name (while typing you need to reach for @ and curly braces), so I find it quite nice to visualize the list of stashes and the diffs in gitg. The annoying part is always choosing the branch where to apply the stash, there's no 'please make the current branch default for stashes'.

### Blame

The client git is pretty straight-forward and has only the ability to blame an entire file. Hence it does `git blame <filename>`. Still useful, though personally I think would be nice to get a link to the commit from the blame section.

![gitg - blaming]({{ site.baseurl }}/assets/img/git/gitg3.png)


### Patch

Very often I find myself willing to add to the staged area only several lines from a file. Then I do `git add -p` and add just the hunks that I want. With gitg I choose the Commit tab and the file I want to patch and simply by holding `CTRL` and clicking on the lines, I can choose exactly what I want to stage. 

The client has this nice feature to stage specific lines, but at the same time it can be quite annoying when you got big chunks of code and you want to make it fast. Some git clients allow to select lines and stage them all at once, unfortunately not gitg.


### Conclusion

At the beginning I though that GUI git is going to be presented as a very handy tool, but eventually I got convinced that it barely associates with the CLI power.
