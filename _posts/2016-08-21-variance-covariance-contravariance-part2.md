---
layout: post
published: true
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
case class List[+T](h: T, t: List[T]) {
  def head: T = h
  def tail: List[T] = t
  def prepend(elem: T): List[T] = List(elem, this)
}
```

The compilers doesn't like the _prepend_ method and gives us `Error: "Covariant type A occurs in contravariant position in type A of value elem"`.

An input parameter being covariant means that it would be bound to a subtype but be invokable with a base type. This is an impossible conversion, because a base type cannot be converted to a subtype.

The way to fix this is using ***lower bounds***:

```scala
case class List[+T](h: T, t: List[T]) {
  def head: T = h
  def tail: List[T] = t
  def prepend[U >: T](elem: U): List[U] =  List(elem, this)
}
```

An this works because 
> _covariant_ type parameters may appear only in ***lower bounds*** of method type parameters

Note that now the _prepend_ method has a slightly less restrictive type: you are allowed to prepend an object of a supertype to an existing list, e.g:

```scala
val chiwawaList: List[Chiwawa] = new List[Chiwawa](new Chiwawa, null)
val mammalList: List[Mammal] = chiwawaList.prepend(new Mammal)
```

-----


Another example of _stupid thing_ is:

```scala
class Kennel[-A] {
	def get: A
}
```

The compilers says `Error: "Contravariant type A occurs in covariant position in type A of value get"`.

Being `Kennel` contravariant means that we could do something like:

```scala
val animalKennel: Kennel[Animal] = new Kennel[Animal]
val dogKennel: Kennel[Dog] = animalKennel
```

If `Kennel[Animal].get` returns instances of `Animal`, are we really sure that we wanto to be able to use it in every place that expects a `Kennel[Dog]`? Nope.

A way to make it compile is 

```scala
class Kennel[-A] {
    def get[B <: A]: B
}
```

An this works because 
> _contravariant_ type parameters may appear only in ***upper bounds*** of method type parameters

-----

Luckily we don't need to remember all the variance position laws, the compiler does all the work for us :)

-----

# *TL;DR*

## Invariant 

```scala
trait Invariant[A] {
  def foo(a: A): Unit
  def baz: A
}
```

- `Invariant[Dog]` has no relation with `Invariant[Animal]`
- Invariant type parameters can be used everywhere


## Covariant

```scala
trait Covariant[+A] {
  def foo[B >: A](b: B): Unit
  def baz: A
}
```

- `Covariant[Dog]` is subtype of `Covariant[Animal]`
- Covariant type parameters can appear:
    - in _lower bounds_ of method type parameters (_foo_)
    - as method results (_baz_)
    - as mutable field type only if the field has object private scope (`private[this]`)
- Generally used in producers (types that return something) and immutable data
    

## Contravariant

```scala
trait Contravariant[-A] {
  def foo[B <: A](): B
  def baz(a: A): Unit
}
```

- `Contravariant[Dog]` is super type of `Contravariant[Animal]`
- Contravariant type parameters can appear:
    - in _upper bounds_ of method type parameters (_foo_)
    - as method arguments (_baz_)
    - as mutable field type only if the field has object private scope (`private[this]`)
- Generally used in consumers (types that accept something)

