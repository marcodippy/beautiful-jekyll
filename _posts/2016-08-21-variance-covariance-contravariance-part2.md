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
Everything makes more sense if you think about automatic type conversions: you can substitute the input parameter _t_ with something with *fewer requirements* (hence more general, a supertype is fine), while the function should return a type *at least as specialised as R* (the caller might expect all the methods on R to be available, a subtype is fine).

Let's try to explain it in other terms: suppose you have a `Function1[Dog, Any]` (Dog => Any), should `Function1[Animal, Any]` be a supertype or a subtype? Remember that the feature available in a type are guaranteed to be available its subtypes, but the contrary is not necessarily true.

`Function1[Dog, Any]` must be a *supertype* of `Function1[Animal, Any]` because if you have a function designed for a `Dog` it will work on Dogs and its subtypes, but not for `Animal`s in general, whereas if you have a generic function for `Animal`s you can always use it for `Dog`s.



### Function parameters are contravariant 
For example, you need a function `val tweet: (Bird => String)`: if you have a function `val sound: (Animal => String)` it's perfectly fine, `Bird` is also an `Animal`

```scala
val tweet: (Bird => String) = ((a: Animal) => a.sound )
```

While 

```scala
val tweet: (Bird => String) = ((c: Chicken) => c.sound )
```

might not work for `Duck` for example.

### Function results are covariant
This is even simpler to demonstrate; assume for a moment that a hierarchy like this exists:

- Animal 
	- Phoenix 
    	- Bird 
    		- Chicken
        	- Duck

you need a function `hatch` to produce a `Bird`... it can produce a `Chicken`, it can produce a `Duck`... should it produce a `Phoenix`??? Nope.

```scala
val hatch: (() => Bird) = (() => new Chicken)
```





