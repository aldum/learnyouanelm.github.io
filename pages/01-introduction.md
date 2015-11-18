---
layout: post
title: Introduction
---

Introduction
============

About this tutorial
-------------------

Welcome to *Learn You an Elm*! If you're reading this,
chances are you want to learn Elm. Well, you've come to the right
place, but let's talk about this tutorial a bit first.

I decided to write this because *Learn You A Haskell* is now
a common resource for learning functional programming. But,
Elm is developing into its own language, and has some significant
technical and philosophical differences from Haskell.
So, instead of throwing away the great resource that is LYAH, we've
dediced to adapt it.

![bird](img/bird.png)

This tutorial is aimed at people who have experience in imperative
programming languages (C, C++, Java, Python …) but haven't programmed in
a functional language before (Elm, F#, Clojure, Scala, Haskell, Scheme, ML, OCaml …). 
Although I bet that
even if you don't have any significant programming experience, a smart
person such as yourself will be able to follow along and learn Elm.

The [elm-discuss mailing list](https://groups.google.com/forum/#!forum/elm-discuss) 
is a great place to ask
questions if you're feeling stuck. People there are extremely nice,
patient and understanding to newbies.

So what's Elm?
------------------

![fx](img/fx.png) Elm is a *functional programming language*, where functions
are *stateless*. In imperative languages you get things
done by giving the computer a sequence of tasks and then it executes
them. While executing them, it can change state. For instance, you set
variable a to 5 and then do some stuff and then set it to something
else. You have control flow structures for doing some action several
times. In stateless functional programming you don't tell the computer what
to do as such but rather you tell it what stuff *is*. The factorial of a
number is the product of all the numbers from 1 to that number, the sum
of a list of numbers is the first number plus the sum of all the other
numbers, and so on. You express that in the form of functions. You also
can't set a variable to something and then set it to something else
later. If you say that a is 5, you can't say it's something else later
because you just said it was 5. What are you, some kind of liar? So in
stateless functional languages, a function has no side-effects. The only
thing a function can do is calculate something and return it as a
result. At first, this seems kind of limiting but it actually has some
very nice consequences: if a function is called twice with the same
parameters, it's guaranteed to return the same result. 
This is a property that makes testing, debugging and refactoring code very easy.
It also makes it easy to build more
complex functions by gluing simple functions together.

Elm is *eager*, unlike Haskell, which is lazy.
That means, in Elm, if you call a function, the arguments are fully evaluated
before they are passed to the function.
Most programming languages are like this, so you won't likely need
to think about this very much.


![boat](img/boat.png) Elm is *statically
typed*. When you compile your program, the compiler knows which piece of
code is a number, which is a string and so on. That means that a lot of
possible errors are caught at compile time. If you try to add together a
number and a string, the compiler will whine at you. Elm uses a very
good type system that has *type inference*. That means that you don't
have to explicitly label every piece of code with a type because the
type system can intelligently figure out a lot about it. If you say a =
5 + 4, you don't have to tell Elm that a is a number, it can figure
that out by itself. Type inference also allows your code to be more
general. If a function you make takes two parameters and adds them
together and you don't explicitly state their type, the function will
work on any two parameters that act like numbers.

Elm is *elegant and concise*. Because it uses a lot of high level
concepts, Elm programs are usually shorter than their imperative
equivalents. And shorter programs are easier to maintain than longer
ones and have less bugs.

Elm was made by Evan Czaplicki, who continues to develop it with
the support of [Prezi, Inc.](https://prezi.com/).
It is currently used in production by several companies,
including [CircuitHub](https://www.circuithub.com/) and  
[NoRedInk](http://tech.noredink.com/).

What you need to dive in
------------------------

Not much. The majority of the code in this book can be compiled and run
using either [elm-lang.org/try](http://elm-lang.org/try),
or [share-elm](http://share-elm.com) (if you want to save your code).

When we get to more complicated examples, you'll need a text-editor,
such as [Atom](https://atom.io/), [Sublime](http://www.sublimetext.com/), 
or [LightTable](http://lighttable.com/),
or any other editor of your choice.
There are lots of [Elm-specific plugins](http://elm-lang.org/install) for editors
that you can download.

You will also need a copy of [The Elm Platform](http://elm-lang.org/install).
If you are on Mac or Windows, use the installers at the above link.
If you are on Linux, you can  install using the [NPM Package](https://www.npmjs.com/package/elm)
Or, you can [build it from source](https://github.com/elm-lang/elm-platform).
