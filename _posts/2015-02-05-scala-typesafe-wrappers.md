---
layout: post
title: "Scala Typesafe Wrappers"
description: "Wrapping simple values to add semantics and typesafety"
category: "Scala"
tags: [scala, type-safety]
---
{% include JB/setup %}

# Why Wrap?

Primitives like Strings, Ints, and Booleans are excellent at holding values, but in most use cases they are weakly typed. Primitives do not carry any semantics or invariants. For example, a string can store an email address but a string _is not_ an email address. If you store an email address in a simple String, any invariants such as 'is it valid?', have to be assumed or checked every time the value is used. This encourages bugs at worst or bloated and overly defensive code at best.

So why are primitives so over used?

* They already exist and are easy to use.
* The cost of using primitives is not paid by original developer who knows where is safe to assume a value is valid and when it must be checked.  However people who maintain or use this code are not so lucky, they have to ensure all preconditions are met before calling the code and don't know when to assume or check any postconditions it guarantees.  

<br/>For a more complete explanation on the ills of primitive typing see this excellent blog [The abject failure of weak typing](http://techblog.realestate.com.au/the-abject-failure-of-weak-typing/).

We can strengthen primitive types by wrapping them with a simple type to encode our semantics and invariants. We verify the invariants only once when we create the value and the compiler proves they are true everywhere we use it.

```scala
class WorkerPool(
  initialSize: PositiveInt = defaultInitialSize,
  maxAllocation: StrictPercentage = defaultMaxAllocation) {
    // ...
  }
```

For example the `WorkerPool` class uses typesafe wrappers to ensure the values passed into the constructor can be used without any extra validation.  Here the compiler ensures that `initialSize` is never negative and that `maxAllocation` is a percentage value between 0 and 100. This simplifies the implementation as `WorkerPool` does not need to validate the values. It also makes the class easier to use as you know exactly what values it supports.

<!--more-->
# Case classes

The simplest way to wrap a value is to use Scala case classes.

```scala
case class Email(value: String) {
  require(Email.isValid(value), s"'$value' must be a valid email address")
}

object Email {
  def isValid(value): Boolean = ???
}

case class PositiveInt(value: Int) {
  require(value >= 0)
}

case class StrictPercentage(value: Int) {
  require(value >= 0 && value <= 100)
}
```

Pros of the case classes approach:

 * Easy to implement and guarantees invariants.
 * Methods like `hashcode`, `equals` and `toString` are implemented by default.
 * A default `apply` method implemented on the companion object.
 * Supports pattern matching

<br/>Cons:

 * Creates an extra object for each value.
 * The `require` method throws exceptions that breaks referential transparency. As a result the user does not know whether they are going to get a value or exception each time they call the constructor.

# Use AnyVal

We can use scala's [AnyVal](http://docs.scala-lang.org/overviews/core/value-classes.html) support to create wrapper types that do not result in an extra object per value.  AnyVals are automatically unboxed and inlined by the compiler, which reduces allocations and load on the garbage collector.

The `hashcode`, `equals` and `toString` methods provided by case classes are easy to implement for a single value.

```scala
trait WrappedValue[T] extends Any {
  def value: T

  override def toString = this.getClass.getName + "(" + value.toString + ")"

  override def equals(other: Any): Boolean = {
    if (this.getClass.isInstance(other)) {
      value.equals(other.asInstanceOf[WrappedValue[T]].value)
      } else {
        false
      }
    }
  }

  override def hashCode: Int = value.hashCode
}

// But AnyVal cannot have a constructor
class PositiveInt(val value: Int) extends AnyVal with WrappedValue[Int] {
  require(value >= 0)  // Does not compile
}
```

# The constructor

We can address the constructor and the exceptions thrown by `require` with a private constructor and a companion object.  The `from` method is now the public constructor for `PositiveInt` which returns a `Try[PositiveInt]`. Now we are using the compiler's type system to capture our invariants and to encode success or failure.

See [The Neophyte's Guide to Scala Part 6: Error Handling With Try](http://danielwestheide.com/blog/2012/12/26/the-neophytes-guide-to-scala-part-6-error-handling-with-try.html) for more information on using `Try`.

```scala
class PositiveInt private (val value: Int) extends AnyVal with WrappedValue[Int]

object PositiveInt {
  def from(value: Int): Try[PositiveInt] = {
    if (value >= 0) {
      Sucess(PositiveInt(value)
    } else {
      Failure(new IllegalArgumentException(s"$value must be positive") with NoStackTrace)
    }
  }

  def unapply(wrapped: PositiveInt): Option[Int] = Some(wrapped.value)
}
```

We also added a simple `unapply` method to the companion object to support case class style pattern matching.

# Companion object

Having to define a custom companion object for every wrapped type adds a lot of boiler-plate, however this can be abstracted into a trait.  Here the `construct` and `validate` methods are abstract as are the `InnerType` and `OuterType` types.

```scala
object WrappedValue {
  trait Companion {
    type InnerType
    type WrappedType <: WrappedValue[InnerType]

    protected def construct(value: InnerType): WrappedType
    protected def validate(value: InnerType): Option[String]

    def from(value: InnerType): Try[WrappedType] = {
      validate(value) match {
        case Some(message) => Failure(new IllegalArgumentException(message) with NoStackTrace)
        case None => Success(construct(value))
      }
    }

    def unapply(wrapped: WrappedType): Option[InnerType] = Some(wrapped.value)
  }
}
```

Each of the companion objects implement these abstract members to describe how to validate and build its type.

```scala
class PositiveInt private (val value: Int) extends AnyVal with WrappedValue[Int]

object PositiveInt extends WrappedValue.Companion {
  type InnerType = Int
  type WrappedType = PositiveInt

  override protected def construct(value: Int): PositiveInt = new PositiveInt(value)

  override protected def validate(value: Int): Option[String] = {
    if (value < 0) Some(value + ": must be positive") else None
  }
}
```

# Ordering

One benefit of primitive types is their support for natural ordering in scala. It is very easy to sort or compare `Int` and `String` values. This natural ordering should extend to our wrapped types too.

The implicit method `ordering` in `WrappedValue.Companion` creates an `Ordering[WrappedType]` if there is an implicit `Ordering[InnerType]`. This means that if `InnerType` can be sorted or compared, so can `OuterType`.  The implementation in `WrappedOrdering` simply delegates to `ord` the `Ordering[InnerType]` implementation.

```scala
import math.Ordering

object WrappedValue {
  trait Companion {
    // ...

    implicit def ordering(implicit ord: Ordering[InnerType]): Ordering[WrappedType] = new WrappedOrdering(ord)

    class WrappedOrdering(ord: Ordering[InnerType]) extends Ordering[WrappedType] {
      override def compare(x: WrappedType, y: WrappedType): Int = {
        ord.compare(x.value, y.value)
      }
    }
  }
}
```

# Unsafe Access

So far the typesafe wrapper is designed for production code.  The only way to construct one is with the `from` method that returns a `Try[WrappedType]`. This makes constructing wrapped values for tests more difficult than it should be.

```scala
"A worker pool" should "accept a default size" in {
  val initialSize = PositiveInt.from(5).get
  val workerPool = new workerPool(defaultSize = initialSize)
  // ...
}
```

As you can see, using the `from` method for a value that cannot fail distracts the test code. It would be nice to enable an unsafe `apply` method on the companion object; one that is only available in tests.

```scala
import PositiveInt.Unsafe._  // Enables the unsafe apply

"A worker pool" should "accept a default size" in {
  val workerPool = new WorkerPool(defaultSize = PositiveInt(5))
  // ...
}
```

This is simple to implement using a typeclass `EnableUnsafe[WrappedType]`.  The typeclass has no body, since its only role is to enable the `apply` method.  As a result, the typeclass is only used at compile time and is a [phantom type](http://james-iry.blogspot.ie/2010/10/phantom-types-in-haskell-and-scala.html).  To enable the `apply` method in tests just import `Unsafe._`.

```scala
trait EnableUnsafe[WrappedType]

object WrappedValue {
  trait Companion {
    // ...

    object Unsafe {
      implicit object Enable extends EnableUnsafe[WrappedType]
    }

    def apply(value: InnerType)(implicit ev: EnableUnsafe[WrappedType]): WrappedType = {
      validate(value) foreach { message => require(false, message) }
      construct(value)
    }
  }
}
```

# It's a Wrap

Typesafe wrappers improve your code's correctness and make it easier to read and  write. Whenever code receives a value from outside, it knows it can use it straight away without any extra defensive checks. The invariants checks are implemented in one place and are only checked when the value is created. This pushes the validation logic to the edge of the system which makes recovery and debugging easier.  It is much easier to handle a bad value when its created than discovering it is bad just before using it.  

# References

* [The abject failure of weak typing](http://techblog.realestate.com.au/the-abject-failure-of-weak-typing/)
* [Value Classes and Universal Traits](http://docs.scala-lang.org/overviews/core/value-classes.html)
* [The Neophyte's Guide to Scala Part 6: Error Handling With Try](http://danielwestheide.com/blog/2012/12/26/the-neophytes-guide-to-scala-part-6-error-handling-with-try.html)
* [Phantom Types In Haskell and Scala](http://james-iry.blogspot.ie/2010/10/phantom-types-in-haskell-and-scala.html)
