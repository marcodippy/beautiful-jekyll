---
layout: post
published: false
title: 'Variance, Covariance and Contravariance - Part 2'
tags:
  - functional-programming
  - variance
  - covariance
  - contravariance
---

So far so good. Now we want a house for our pet; I can imagine that a generic kennel for `Dog` can be fine for `Chiwawa` and `Bulldog` as well.

```scala
class Kennel[+A] {
  def put(animal : A) : Kennel[A] 
  //Error: "Covariant type A occurrs in contravariant position in type A of value animal"
}
```

Unfortunately this doesn't work and the error is quite cryptic. But this make a lot of sense, especially if we take a look at how `Function1` is defined in Scala:

```scala
trait Function1[-T1, +R] {
	def apply(t : T1) : R
}
```

Function1 is _contravariant_ on its argument and covariant on its result (in general input parameters are contravariant and results are covariant).
Everything makes more sense if you think about automatic type conversions: you can substitute the input parameter _t_ with something with *fewer requirements* (hence more general), while the function should return a type *at least as specialised as R* (the caller might expect all the methods on R to be available).


