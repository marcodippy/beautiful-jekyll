---
layout: post
title: First post!
tags:
  - random
  - exciting-stuff
published: false
---

The first key concept you'll encounter if you start reading about _functional programming_ is ****pure function****.

A function is pure if:
- its result is ****always**** the same given the same input ( A=>B is pure if it relates every _a_ of type A with exactly one _b_ of type B such that _b_ is determined solely by the value of _a_ )
- it has ****no side effects****

_Side Effects???_ If a function does something more than merely returning a result, all this stuff in excess is side effect: throwing an exception, performing I/O, setting a variable, modifying some kind of state in general... 

The absence of side effects in our functions make them more general, easier to test, reuse, parallelize and less prone to bugs. 


## > Pure functions are easier to reason about.

You can ****trust**** pure functions. They are ****transparent****, there's nothing hidden inside them, everything a function does is represented by the value that it returns.

Think about the _plus_ function: would you ever doubt that 2 + 3 = 5 ?
_plus_ is a pure function, we can trust it, so we are allowed to solve this piece of code like it was an algebraic equation:


```scala
val a = 2
val b = a + 3
val c = b + 5 // c = 10, do you agree?
```

Step 1: replace _a_
```scala
val a = 2
val b = 2 + 3
val c = b + 5
```

Step 2: replace _b_
```scala
val a = 2
val b = 2 + 3
val c = 2 + 3 + 5 // c = 10
```

We can see that the result _c_ is still the same.

The typical counterexample is done with the nasty _append_ function from the `java.lang.StringBuilder` class.


