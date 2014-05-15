---
layout: post
title: "Scala For Beginners"
description: "Converting Java to Scala Developers"
category: "Scala"
tags: [scala]
---
{% include JB/setup %}

In Workday we've been using Scala for a couple of years now. The availability of engineers with Scala experience is getting better: starting from zero the only way is up! Most of the time we try to help experienced Java developers become Scala developers. Here's how we approach the problem:

## Welcome to a better world ## 

The first thing we do is get developers excited about why we like Scala so much. There are many reasons: 

* Succint code / Less boilerplate -  less code means less maintenance.
* It's Java-like and supports OO - so the transition can be gradual.
* Powerful type sytem - let the compiler help you write statically correct code. 
* Collections API -  aka no more nested for loops for you.  Map, filter and reduce like a pro.
* Closures - Java 8 will have some support, but you get that and more right now in Scala.  
* Functional Programming (FP) - there's more to FP than just collections. 
* DSL's support - create internal DSL's and use them directly in your code. 
* Concurrency - Scala's Futures make Java Future's look pre-historic.
  
<p></p> 
  
While these are all good reasons, we've noticed a tangible change in developers when they start down the Scala path that can be summarised as: 

> Learn to love your code (again)

<!--more-->  
We've all struggled with masses of code. It's a drag. Coding in Scala can be fun. Well thought out Scala code allows you to boil away a lot of this kruft and define code which aims to clearly represent the intention of the developer (and nothing more). In practice this means developers, iteratively refine their Scala code - it becomes a kind of game. 

## Novice Stage ## 

Everyone has to start somewhere. We've found that getting [training really helps](http://typesafe.com/how/training). This is especially important if you are trying to transition your team over to Scala - once code goes into production, everyone on your team needs to understand Scala, or they can't support that code. 

Try attending [Scala Meetups](http://scala.meetup.com/). If there's one in your area, it is a good way to meet like-minded developers and ask questions.

Start Collecting Scala links, tips and idioms. This helps to build a resource for new learners. To illustrate, here's our one:  

* [Workday's Scala Page](/pages/Scala/index.html). 

You aren't the first person to ask Question X - [Stack Overflow](http://stackoverflow.com/) usually has the answer. 

Skirt around / willfully ignore the difficult areas - concentrate on learning the syntax: closures, partial functions, case matching and so on. Have a look at the [Basic Scala Examples](http://www.scala-lang.org/old/node/220.html).

Learn to love the REPL - it's the Scala Interpreter. You can use it to try out your newly learned [syntax](http://www.javacodegeeks.com/2011/09/scala-tutorial-scala-repl-expressions.html) - it's your best friend.  

## Advanced Beginner Stage ##

Now it's time to start using Scala in your daily programming life. You should now understand most of the main [Scala syntax](http://www.scala-lang.org/old/node/44.html). That means committing code. How to do this without crashing in a ball of flames? Start with adding unit-tests to existing Java code. Our first Scala code was certainly not a pretty as our current code. Unit-tests provide a safe way for you to start coding daily in Scala. This will help your learning process immensely. It also gives you a way to start really evaluating your switch:  are the tests easier to write, do they more clearly represent intent? 

Develop an opinion on the semi-religious question of which unit-test framework you should use: [ScalaTest](http://www.scalatest.org/) vs [Specs2](http://etorreborre.github.io/specs2/). 

Load-tests are another good place to start writing Scala code. While tools such as [Gatling](http://gatling-tool.org/) and [Jmeter](http://jmeter.apache.org/) exist for testing Web Services, we often have needs for load-testing individual modules. Scala's closures, collections and futures provide a way to abstract some of this behaviour. For inspiration - [see here](http://www.scala-blogs.org/2008/12/growing-language.html)

Start developing your own common library for use in your Scala projects. As you learn new Scala patterns and idioms and embed them in your new library, take time to document them (scaladoc) in code. You may understand the idiom, but other's might not. Once it is there is must be supported. 


## Competent Stage ##

At this stage you should know much of the syntax and be getting ready to put code into production. More importantly you'll have to support and maintain this code - your safety net has been removed. A key thing to understand now are the performance aspects of your code. Read Josh Sureth's book [Scala in Depth](http://www.manning.com/suereth/), then read it again. It's a critical resource. For instance, you should understand the memory differences between Lists, Streams and Iterators and make heavy use of the @tailrec annotation.  

You hold code-reviews right? Well you should also hold regular Scala brown-bags. Scala's learning curve can be steep. We all learn at different rates and learn different things based largely on what we are currently working on. New developers will arrive that don't have the same knowledge as you. Don't hide your hard-won Scala skills, share them! That way everyone benefits. Continual learning: what's new, what works for you, what doesn't. 

Do the [Functional Programming Principles in Scala](https://www.coursera.org/course/progfun) course from Coursera. One of the advantages of Scala is it allows you to mix OO and FP styles together easily. In practice this means much of your early code is OO-style - since that's what you know from Java. This course helps get your head moving towards programming in a more FP-style. 

Start learning the tricks of DSL design. Scala was designed to be extensible. The core language spec is pretty small and most of the things you think about as part of the language are actually DSL's built on the core language spec. 

## Proficient Stage ## 

Learn [Akka](http://akka.io/) - we are all distributed / parallel programmers now. Threads and locks in Java are all well and good, but they can in practice be difficult to reason about and that means difficult to support and maintain. Akka, based on the actor model is quickly becoming a kind of killer app / library for handing distribution problems. It can make your systems easier to build, reason about and support. Warning: Akka is not just a library, it's a way of life. 

Start getting to grips with the trickier functional aspects of Scala, e.g. Monads, Higher Kinded Types and such. At this stage you should be capable of building new libraries in Scala and to do this well, means mastering the more complicated concepts.  

The pace of development in Scala is fast - much faster than Java. [Scala Days](http://scaladays.org/) is the central language conference - we like it, but how best to put this? Some talks can be somewhat more geared towards academic / language nerds. That said much of the content is online and the keynotes are definitely worth your time.  We also liked [Skillsmatter Scala 2012](http://skillsmatter.com/event/scala/scala-exchange-2012).

## Expert Stage ## 

By now you are master of FP. You can make interesting comparisons between Scala, Scalaz, Haskell, OCaml etc. You're probably have your own or are heavily involved in a [Scala Improvement Process (SIP)](http://docs.scala-lang.org/sips/). There are rumours you may also be able to hover at will. 

