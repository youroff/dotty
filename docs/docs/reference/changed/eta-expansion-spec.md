---
layout: doc-page
title: "Automatic Eta Expansion - More Details"
---

### Motivation

Scala maintains a convenient distinction between _methods_ and _functions_.
Methods are part of the definition of a class that can be invoked in objects while functions are complete objects themselves, making them first-class entities. For example they can be assigned in variables.
These two mechanisms are bridged in Scala by a mechanism called _eta-expansion_ in literature also called eta-abstraction).
According to this, methods can be turned into functions.
The intuition behind this, is that if we have a function `f(x)` and we need to pass it around
we can either pass its name `f` or a function `x => f(x)` which expresses the idea that two functions
are equivalent if and only if they give the same result for all arguments.

Consequently, the essense of eta-expansion is captured in the following snippet.
Imagine that the `val` is generated by the compiler, when the programmer writes ```f = m```.
The right-hand side is not a function so the compiler performs _automatic eta-expansion_:

```scala
def m(x: Int, y: String) = ???
val f = m // generates val f = (x: Int, y: String) => m(x, y)
```

In Scala, previously, a method reference `m` was converted to a function value
only if the expected type was a function type. If that was not the
case, one had to write `m _` to force the conversion.

For methods with one or more parameters like in the example above, this restriction has now been
dropped. The syntax `m _` is no longer needed and will be deprecated in the
future.

## Automatic eta-expansion and partial application
In the following example `m` can be partially applied to the first two parameters.
Assignining `m` to `f1` will automatically eta-expand.

```scala
def m(x: Boolean, y: String)(z: Int): List[Int]
val f1 = m
val f2 = m(true, "abc")
```

This creates two function values:

```scala
f1: (Boolean, String) => Int => List[Int]
f2: Int => List[Int]
```

## Automatic eta-expansion and implicit parameter lists

Methods with implicit parameter lists will always get applied to implicit arguments.

```scala
def foo(x: Int)(implicit p: Double): Float = ???
implicit val bla: Double = 1.0

val bar = foo // val bar: Int => Float = ...
```

## Automatic Eta-Expansion and implicit function types

Methods with implicit parameter lists can be assigned to a value with an implicit function type
only by using the expected type explicitly.

```scala
def foo(x: Int)(implicit p: Double): Float = ???
val bar: implicit Double => Float = foo(3) // val bar: implicit Double => Float = ...
```

## Rules

- If `m` has one or more parameters, we always eta-expand
- If `m` is nullary (i.e. has type `()R`):
    1. If the expected type is of the form `() => T`, we eta expand.
    2. If m is defined by Java, or overrides a Java defined method, we insert `()`.
    3. Otherwise we issue an error of the form:
Unapplied nullary methods are only converted to functions when a function type is expected.
You need to either apply the method to `()`, or convert it to a function with `() => m()`.

The syntax `m _` is deprecated.

### Reference

For more info, see [PR #2701](https://github.com/lampepfl/dotty/pull/2701).

