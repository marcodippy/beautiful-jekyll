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

To better understand how variance works, let's inspect the definition of the trait `Function1` 

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

## The Scala compiler is our friend
Working with variance annotation can be tricky, but fortunately the compiler is our ally and it prevents us to make stupid things. An example of _stupid thing_ is:

```scala
class Kennel[+A] {
	def put(animal: A): Unit = ???
}
```

The compilers doesn't like that and gives us `Error: "Covariant type A occurs in contravariant position in type A of value animal"`.

Remember, that being Kennel covariant means that we could do something like this:

```scala
val dogKennel: Kennel[Dog] = new Kennel[Dog]
val animalKennel: Kennel[Animal] = dogKennel
```

If the _put_ method can operate on `Dogs` does it make sense (in general) that it should also work on `Animals`? If so, we should generalize the Kennel class by using *contravariance*, otherwise the way to fix this is using ***lower bounds***:

```scala
class Kennel[+A] {
	def put[B >: A](b: B): Unit = ???
}
```
An this works because of two basic rules of variance:

- _covariant_ type parameters may appear only in ***lower bounds*** of method type parameters
- _contravariant_ type parameters may appear only in ***upper bounds*** of method type parameters


Another example of _stupid thing_ is:

```scala
class Kennel[-A] {
	def get: A
}
```

The compilers says `Error: "Contravariant type A occurs in covariant position in type A of value get"`.

Being Kennel contravariant means that we could do something like:

```scala
val animalKennel: Kennel[Animal] = new Kennel[Animal]
val dogKennel: Kennel[Dog] = animalKennel
```

Again, if Kennel[Animal].get returns instances of the Animal class, are we really sure that we wanto to be able to use it in every place that expects a Kennel[Dog]? Nope.

## Variance Positions
So, Scala compiler enforces some rules one the variance annotations position, the foundamental ones are:

- _covariant_ type parameters can appear only in method results
- _contravariant_ type parameters can appear only in method parameters
- _invariant_ type parameters can appear everywhere

Luckily the compiler does all these checks for us and we don't have to remember all the laws :)

## Generic Rules
Covariancy
- for immutable (typically), but mutable cannot



