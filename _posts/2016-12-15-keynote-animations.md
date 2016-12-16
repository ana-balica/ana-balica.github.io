---
layout: post
title: "Keynote animations: express yourself"
date:   2016-12-15
tags:
  - keynote
  - tools
---

Keynote might not be the best tool to create animations,
but it's good enough and provides most common animation tools.
Moreover you don't really want to open After Effects and export a movie
just to make an object wiggle a bit during your presentation.
It's also an interesting challenge to create something complex
out of something trivial.
Constraints foster creativity.

### Meaning

The main goal of an animation is to convey meaning.
Think of it as another dimension to express an idea.
Try to remove the animation and see if you can communicate
the same message equally well.
If you loose clarity, then this is a meaningful animation.

<video autoplay loop>
  <source src="{{ site.baseurl }}/assets/video/keynote/potato.mp4" type="video/mp4">
</video>

In this case, the animation is used to convey playfulness.
"Potato is a serious company, but we also like to be silly."

### Don't overdo it

In conjunction with the previous statement, don't fill up your presentation
with useless animations.
When things move, spin, burn too often, it's hard to follow along
and justify any of this live action on the screen.

On a more practical note, certain types of animations might even make some people dizzy -
I'm looking at you [Prezi](https://prezi.com/_ueivw8ad8xx/prezi-demo/).

<video autoplay loop>
  <source src="{{ site.baseurl }}/assets/video/keynote/tacky.mp4" type="video/mp4">
</video>

Oh you, flame effect, I will never use you, unless I'm going for on-purpose tacky style.

### Baby steps

So how to make the first slide with the smiling Potato?
First solution: use 2 slides, one with a plain Potato
and another with a Potato with a big smile on it.
Better solution: apply an animation called "Appear",
found in the "Build In" tab of the Animate section.
Click/tap and, voila, the image appears!

<div class="video-wrapper">
  <iframe src="https://www.youtube.com/embed/W3naF-73Olw" frameborder="0" allowfullscreen></iframe>
</div>

### Bullet points

They are haunting us in every presentation! They are useful though, as they reduce
long complex sentences to a few important key points.
And if you want to make them appear in a special way, say one by one,
Keynote has got your back.

<div class="video-wrapper">
  <iframe src="https://www.youtube.com/embed/vD5s2_5qVf8" frameborder="0" allowfullscreen></iframe>
</div>

Don't even think making 5 separate slides and placing on each plus-one bullet point.
This easily gets out of sync when you decide to add or remove a bullet point
and now you have to edit not one slide, but 5 (worst-case scenario).

These is an exception to the rule though.
If the speaker notes for the first 2 points are already too big
and in presentation mode you need to scroll to get to the last sentence,
then it's a good idea to split up the notes into a couple of slides
and hence duplicate the content. *I should start doing that* ʕ⊙ᴥ⊙ʔ

### Center of attention: highlighting

As you speak you might want to direct the attention of your audience
to a specific line of code, element or word.
This is a neat trick, because it allows the viewers to grasp the general picture at first
and then dive deep into certain segments.
There are many ways to do that.
In case of code I like to use highlighting.

<div class="video-wrapper">
  <iframe src="https://www.youtube.com/embed/8KwH_ZmM6Co" frameborder="0" allowfullscreen></iframe>
</div>

Create a shape and give it enough opacity to make the text underneath eligible.
Then apply an appropriate "Build In" animation to the shape -
I like "Move In" with a little bit of bounce.
It's hard to ignore such an animation, and at the same time it doesn't make me cringe
(like "Bouncy").

**Tip:** assess animation duration.
The animation should be fast and noticeable.
In most cases 1s is too long.
Experiment with 0.5 - 1s range and see what feels best to your presentation pace.

### Center of attention: opacity

Similarly to highlighting, you can use opacity to make something stand out
in the midst of blurred/semi-transparent content.
I remember seeing the trick for the first time at DjangoCon Europe 2015 in Cardiff
in Ola's talk [Pushing the pony's boundaries](https://vimeo.com/channels/952478/134817269)
and I really liked it.

<div class="video-wrapper">
  <iframe src="https://www.youtube.com/embed/fXgcwCpiBQI" frameborder="0" allowfullscreen></iframe>
</div>

"Opacity" animation can be found under the "Action" tab.
Since I prefer to almost omit the transition of going transparent,
I set the duration to 0.1s.

An important take-away from this short video is the "Build Order" dialog.
This is the control panel of all animations, where they can be deleted,
grouped and rearranged.

By the way, alternatively to copy-pasting animations, it's also possible to group elements together
and apply a single animation.
Unfortunately for the current 3 elements when grouped, the animation doesn't apply well.
Would be fun to aggregate and post all Keynote bugs I've discovered while preparing for my talks.

### Multi-actions

Things get complicated when you want something to go opaque, then transparent,
then opaque again.
In other words, apply multi-actions to the same element.
It's somewhat simple - click "Add Action", select another animation and configure it.
My problem with that is the UI that is showing only the last applied action.
To be able to go to older layers of animation, you need to open "Build Order"
and operate from there (that was so not obvious at the beginning).

<div class="video-wrapper">
  <iframe src="https://www.youtube.com/embed/-CFr_VluCjg" frameborder="0" allowfullscreen></iframe>
</div>

### A process

A single animation is worth a thousand words when modeling a process.
It's like when they spend 5 pages to explain how an engine works,
and you still don't get it.
But once you see the animation, it suddenly becomes clear.

<div class="video-wrapper">
  <iframe src="https://www.youtube.com/embed/EQzfc7OnuMY" frameborder="0" allowfullscreen></iframe>
</div>

Here I explain visually how tests are run in parallel, simplified version taken from my
[Testing in Django](https://www.youtube.com/watch?v=EHyKzPQFXzo)
talk at Django Under The Hood 2016.
This is a showcase of what is possible and how it can be achieved using
Keynote animation toolkit.

-----

Now you are ready to unleash your creative freedom and deliver
jaw-dropping presentations. *No flames please.*
