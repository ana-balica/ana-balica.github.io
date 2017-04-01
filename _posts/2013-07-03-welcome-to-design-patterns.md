---
layout: post
title:  "Welcome to Design Patterns"
date:   2013-07-03
tags:
  - code
---

There are many recommended books and articles to be read on Design Patterns. High variety leads to a problem of choice. So based on the magic of rational randomization I decided to read and **understand** [Head First Design Patterns](http://www.amazon.com/First-Design-Patterns-Elisabeth-Freeman/dp/0596007124). I am a Head First virgin, so I even read the pre-introduction with explanations on metacognition, pretty pictures and redundancy. So far, so good.

The reason of writing all this down is to overflow my brain with redundancy, "do my homework" by summarizing everything read in a chapter and maybe leave a note for future me and all other Internet readers about something important. Moreover that's going to be a shortcut on the book (though the book is funny).

One chapter &#8594; one article. And the first one is called *Welcome to Design Patters*.

## Encapsulation

The first announced Design Principle in the book is:

> Identify the aspects of your application that vary and separate them from what stays the same.

That is the guru of all Design Patterns. That means that all patterns are trying to accomplish this in some way or another. It is very important to separate what varies from what stays the same, since changes are too frequent in IT world.

The examples in the book are based on ducks. We will examine a very similar situation based on the Design Puzzle. Initially we have a very simple application that is made of a simple character hierarchy.

![UML simple inheritance]({{ site.baseurl }}/assets/img/design-patterns/uml_simple_inheritance.png)

That's a very inflexible hierarchy and will lead to many problems over time:

1. What if Queen doesn't fight? Should you overwrite the method in this class?
2. Are there going to be more characters that don't fight? Will you overwrite the behaviour in every class?
2. Trolls fight different from Kings!

Ok, that's the moment when you should `SHIFT+Delete` your code. Kidding :) You need to apply that one first principle and conceptually do the following:

![Encapsulation]({{ site.baseurl }}/assets/img/design-patterns/encapsulation.png)

## Interface

That green spot represents all the `Fight/Weapon` Behaviours. It's not a good idea to create just a bunch of classes, since you will lose the ability to use polymorphic behaviour and will hard-code each behaviour to each character. Create an interface. Side-note: here the interface doesn't mean necessarily a Java interface - it's a conceptual interface, a special class that holds the public declarations and has no proper implementation (also not always true, think about abstract classes in C++ which can contain implementation).

> Program to an interface, not to an implementation.

![Interface]({{ site.baseurl }}/assets/img/design-patterns/interface.png)

Here's what we have done after learning that interfaces give you lots of flexibility. In your Character class you will create a property of type `WeaponBehaviour`. Here, let me show you the code.

{% highlight java %}
public abstract class Character {
	WeaponBehaviour weaponBehaviour;

	public void setWeaponBehaviour(WeaponBehaviour w) {
		this.weaponBehaviour = w;
	}

	public void fight() {
		weaponBehaviour.useWeapon();
	}
}

public interface WeaponBehaviour {
	public void useWeapon();
}

public class KnifeBehaviour implements WeaponBehaviour {
	public void useWeapon() {
		// an implementation of a fight with a knife
	}
}

public class Queen extends Character {
	public Queen() {
		this.weaponBehaviour = new KnifeBehaviour();
	}
}
{% endhighlight %}

Moreover by having our `WeaponBehaviour` setter, we can successfully teach our `Characters` to learn how to use new weapons at runtime.

## Composition

A perfect inheritance is a IS-A relationship. It is actually more than just a IS-A according to L principle from SOLID. Nevertheless what we have done here is eliminating IS-A and using HAS-A, which is composition.

> Favor composition over inheritance.

It gave us all those benefits of **behaviour change at runtime** and the encapsulation of a set of algorithms. Now not only `Character` descendants will be able to fight and use weapons, but also any other class can be taught by the power of composition.

By the way, all that talk about fighting characters is a patters called **Strategy**.

> The Strategy Pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from client that use it.

Strategy is a behavioural pattern. If you want to learn more, [check this out](http://www.vincehuston.org/dp/strategy.html).

## Conclusion

![Patterns rule]({{ site.baseurl }}/assets/img/design-patterns/patterns_rule.jpg)
