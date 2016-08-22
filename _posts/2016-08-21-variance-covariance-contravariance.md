---
layout: post
published: true
title: 'Variance, Covariance and Contravariance'
tags:
  - functional-programming
  - variance
  - covariance
  - contravariance
---

Everytime I read something about variance I get confused. It's worth reorganising some concepts and write them down in a blog post!

Let's start by creating a hierarchy of classes:

```scala
class Animal
class Dog extends Animal
class Chiwawa extends Dog
class Bulldog extends Dog
```

The easiest concept to explain is ***invariance***

```scala
class InvariantLeash[A]

def foo(x : InvariantLeash[Dog]) : InvariantLeash[Dog] = identity(x)

foo(new InvariantLeash[Animal])  // type error
foo(new InvariantLeash[Dog]) 	 // success
foo(new InvariantLeash[Chiwawa]) // type error
foo(new InvariantLeash[Bulldog]) // type error
```

We can see that there is no relation between `InvariantLeash[Dog]` and `InvariantLeash[Animal]` or between `InvariantLeash[Dog]` and `InvariantLeash[Chiwawa]`.
The leash can be used only for the specific class it's been created for, in this case `Dog` (pretty useless in this case).

## Covariant type parameters can automatically morph into one of their base types when necessary.

Our chiwawa is getting old, maybe is time to get a more generic leash that can be used also for other dog breeds.

```scala
class CovariantLeash[+A]

def foo(x : CovariantLeash[Dog]) : CovariantLeash[Dog] = identity(x)

foo(new CovariantLeash[Animal])  // type error
foo(new CovariantLeash[Dog])     // success
foo(new CovariantLeash[Chiwawa]) // success
foo(new CovariantLeash[Bulldog]) // success
```

Great, this new leash can be used with `Dog`, `Chiwawa` and `Bulldog`, meaning that 
`CovariantLeash[Chiwawa]` and `CovariantLeash[Chiwawa]` are now subtypes of `CovariantLeash[Dog]`.
This is ***covariance***.

## Contravariance is where a type parameter may morph into a subtype.

I love my pets, I don't want my chiwawa and my bulldog to be leashed, but I still want to own a leash for dogs and animals in general. Let's buy a ***contravariant*** leash!

```scala
class ContravariantLeash[-A]

def foo(x : ContravariantLeash[Dog]) : ContravariantLeash[Dog] = identity(x)

foo(new ContravariantLeash[Animal])  // success
foo(new ContravariantLeash[Dog])     // success
foo(new ContravariantLeash[Chiwawa]) // type error
foo(new ContravariantLeash[Bulldog]) // type error
```



So far so good. Now we want a house for our pet; I can imagine that a generic kennel for `Dog` can be fine for `Chiwawa` and `Bulldog` as well.

```scala
class Kennel[+A] {
  def put(animal : A) : Kennel[A] 
  //Error: "Covariant type A occurrs in contravariant position in type A of value animal"
}
```

Unfortunately this doesn't work and the error is quite cryptic. A bit deeper dive into variance is necessary to understand this dark matter, if you're interested you can find the second part of this post here.
