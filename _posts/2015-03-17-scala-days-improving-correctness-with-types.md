---
layout: post
title: "Scala Days 2015 - Improving Correctness with Types"
description: ""
category:
tags: [events, scala, correctness, scala-days, iain-hull]
---
{% include JB/setup %}

Iain Hull will be presenting at [Scala Days SF 2015](http://event.scaladays.org/scaladays-sanfran-2015#!#schedulePopupExtras-6553) 

# Abstract

This talk is aimed at Scala developers with a background in object oriented programming who want to learn new ways to use types to improve the correctness of their code. It introduces the topic in a practical fashion, concentrating on the “easy wins” developers can apply to their code today.

<a href="http://www.slideshare.net/IainHull/improving-correctness-with-types">
![Slides](/assets/scala-days-improving-correctness-with-types/thumbnail.png)
</a>

[View slides online](http://www.slideshare.net/IainHull/improving-correctness-with-types)

# References

## Inspiration

The following links inspired this talk:

* [The abject failure of weak typing](http://techblog.realestate.com.au/the-abject-failure-of-weak-typing/)
* [Using the right type](http://like-a-boss.net/2014/05/11/using-the-right-type.html)
* [Strong types and their impact on testing](http://levinotik.com/strong-types-and-their-impact-on-testing/)
* [Type Driven Development](http://www.slideshare.net/mproch/booster-conf?next_slideshow=1)
* [More Typing, Less Testing: TDD with Static Types, Part 1](http://spin.atomicobject.com/2014/12/09/typed-language-tdd-part1/)
* [More Typing, Less Testing: TDD with Static Types, Part 2](http://spin.atomicobject.com/2014/12/10/typed-language-tdd-part2/)
* [Types vs. Tests: An Epic Battle?](http://www.infoq.com/presentations/Types-Tests)

<!--more-->

## Defensive programming

* [DefensiveProgramming c2.com](http://c2.com/cgi/wiki?DefensiveProgramming)

## Fail fast

* [OffensiveProgramming c2.com](http://c2.com/cgi/wiki?OffensiveProgramming)
* [Fail fast - Martin Fowler](http://www.martinfowler.com/ieeeSoftware/failFast.pdf)

## Design by contract

* [Building bug-free O-O software: An introduction to Design by Contract](https://archive.eiffel.com/doc/manuals/technology/contract/)
* [Java Modelling Language](http://www.eecs.ucf.edu/~leavens/JML/index.shtml)
* [Design by Contract with JML](http://www.jmlspecs.org/jmldbc.pdf)
* [Beyond Assertions: Advanced Specification and Verification with JML and ESC/Java2](http://www.eecs.ucf.edu/~leavens/JML/fmco.pdf)
* [Spec#](http://research.microsoft.com/apps/mobile/showpage.aspx?page=/en-us/projects/specsharp/)

## Never use nulls

* [Null references, my billion dollar mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)
* [The Neophyte's Guide to Scala Part 5: The Option Type](http://danielwestheide.com/blog/2012/12/19/the-neophytes-guide-to-scala-part-5-the-option-type.html)
* [scala.Option Cheat Sheet](http://tonymorris.github.io/blog/posts/scalaoption-cheat-sheet/)
* [ScalaZ Validation](http://eed3si9n.com/learning-scalaz/Validation.html)

## Almost all data is immutable

* [Scala idiom: Prefer immutable code (immutable data structures)](http://alvinalexander.com/scala/scala-idiom-immutable-code-functional-programming-immutability)

## Do not throw exceptions

* [The Neophyte's Guide to Scala Part 6: Error Handling With Try](http://danielwestheide.com/blog/2012/12/26/the-neophytes-guide-to-scala-part-6-error-handling-with-try.html)
* [Try, option or either](http://blog.xebia.com/2015/02/18/try-option-or-either/)
* [Error handling without throwing your hands up](http://underscore.io/blog/posts/2015/02/13/error-handling-without-throwing-your-hands-up.html)
[Error handling (Typelevel)](http://typelevel.org/blog/2014/02/21/error-handling.html)
* [Scalactic: Or and Every](http://www.scalactic.org/user_guide/OrAndEvery)
* [Comparing Functional Error Handling in Scalaz and Scalactic](https://thenewcircle.com/s/post/1704/comparing_functional_error_handling_in_scalaz_and_scalactic)

## Wrapper types

* [Value Classes and Universal Traits](http://docs.scala-lang.org/overviews/core/value-classes.html)
* [Scala Typesafe Wrappers](http://workday.github.io/scala/2015/02/05/scala-typesafe-wrappers/)

## Type safe equals

* [Scalactic: Constrained equality](http://www.scalactic.org/user_guide/ConstrainedEquality)
* [Learning Scalaz: Day 1](http://eed3si9n.com/learning-scalaz-day1)
* [Implicits Unchained – Type-safe Equality – Part 1](http://hseeberger.github.io/blog/2013/05/30/implicits-unchained-type-safe-equality-part1/) [Part 2](http://hseeberger.github.io/blog/2013/05/31/implicits-unchained-type-safe-equality-part2/) [Part 3](http://hseeberger.github.io/blog/2013/06/01/implicits-unchained-type-safe-equality-part3/)
 
## Non Empty List 

* [scalaz.NonEmptyList](http://docs.typelevel.org/api/scalaz/nightly/index.html#scalaz.NonEmptyList)
* [Higher Kinded Tripe - Non-empty lists](https://higherkindedtripe.wordpress.com/2012/03/10/non-empty-lists/)
* [debasishg NonEmptyList](https://twitter.com/debasishg/status/567369365977190402)
* [Scalactic Every](http://doc.scalatest.org/2.2.4/org/scalactic/Every.html)


## Algebriac data types

* [Effective Scala: Case classes as algebraic data types](http://twitter.github.io/effectivescala/#Functional programming-Case classes as algebraic data types)
* [What the Heck are Algebraic Data Types? ( for Programmers )](http://merrigrove.blogspot.de/2011/12/another-introduction-to-algebraic-data.html)

## Tagged types

*	[Practical uses for Unboxed Tagged Types](http://etorreborre.blogspot.ie/2011/11/practical-uses-for-unboxed-tagged-types.html)
*	[Unboxed Tagged Angst](http://noelwelsh.com/programming/2014/01/29/unboxed-tagged-angst/)
*	[Shapeless implementation - Object tag](https://github.com/milessabin/shapeless/blob/master/core/src/main/scala/shapeless/typeoperators.scala)
*	[Typelevel hackery tricks in Scala](http://www.folone.info/blog/Typelevel-Hackery/)


## More Advanced types

Things I didn't get a chance to cover, but you might be interested in.

###	Path dependent types

* [The Neophyte's Guide to Scala Part 13: Path-dependent Types](http://danielwestheide.com/blog/2013/02/13/the-neophytes-guide-to-scala-part-13-path-dependent-types.html)

### Self-Recursive Type (F-Bounded Types)

* [Recursive Type Signatures in Scala](http://blog.originate.com/blog/2014/02/27/types-inside-types-in-scala/)
* [F-Bounded Polymorphism: Recursive Type Signatures in Scala](https://thenewcircle.com/s/post/1717/f_bounded_polymorphism_marconi_lanna_video)
* [Scala Type of Types - Self-Recursive Type](http://ktoso.github.io/scala-types-of-types/#self-recursive-type)

###	Phantom types

* [Type safe builder in Scala using type constraints](http://www.tikalk.com/java/type-safe-builder-scala-using-type-constraints/)
* [Type-safe Builder Pattern in Scala](http://blog.rafaelferreira.net/2008/07/type-safe-builder-pattern-in-scala.html)
* [Statically Controlling Calls to Methods in Scala](http://www.blumenfeld-maso.com/2011/05/statically-controlling-calls-to-methods-in-scala/)
* [Going Rogue, Part 2: Phantom Types](http://engineering.foursquare.com/2011/01/31/going-rogue-part-2-phantom-types/)

### Type level programming

* [Demystifying Shapeless](http://www.slideshare.net/roeschinc/scala-bythebay)
* [Type level programming in Scala]( https://apocalisp.wordpress.com/2010/06/08/type-level-programming-in-scala/)
* [Type programming shifting from values to types]( http://proseand.co.nz/2014/02/17/type-programming-shifting-from-values-to-types/)

### Shapeless Dependent types and DbC 

* [Introduction to Dependent Types in Scala](https://www.hakkalabs.co/articles/introduction-dependent-types-scala) ([slides](http://wheaties.github.io/Presentations/Scala-Dep-Types/dependent-types.html#/))
* [Unifying Programming and Math – The Dependent Type Revolution](http://spin.atomicobject.com/2012/11/11/unifying-programming-and-math-the-dependent-type-revolution/)
* [Dependent types and Design by Contract](https://groups.google.com/forum/#!topic/shapeless-dev/noxeeUVIgv8)
* [Relationship between contracts and dependent typing](http://cstheory.stackexchange.com/questions/5228/relationship-between-contracts-and-dependent-typing)

# About Iain

Iain Hull [@IainHull](http://twitter.com/IainHull) is a
Senior Software Developer @ Workday and is a member of the Grid Team.
The Grid Team use Scala, Akka, Play and Spray to provide Workday's elastic job
execution evironment.

