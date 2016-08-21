---
layout: post
published: false
title: 'Variance, Covariance and Contravariance'
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

We can see that there is no relashion between InvariantLeash[Dog] and InvariantLeash[Animal] or between InvariantLeash[Dog] and InvariantLeash[Chiwawa].
The leash can be used only for the specific class it's been created for, in this case `Dog` (pretty useless in this case).


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
`CovariantLeash[Chiwawa]` and `CovariantLeash[Chiwawa]` are now subtypes of `CovariantLeash[Dog]`



