---
title: "Kotlin"
date: 2022-09-27T13:27:40+02:00
draft: true
---

Kotlin is a language developed by Jetbrains that can be compiled to the JVM,  and can interoperate with Java libraries.
We use it extensively to develop our Java applications.

It provides a lot of features already present in Scala,  and other languages,  but reduces the learning curve.

# Things we heavily build opon
## Null safety

Kotlin per default doesn't allow types to be *null*.
This is a great feature,  and prevents the dreaded *NullPointerException*.
Types can be made nullable by appending a question mark to the type.
When a type is made nullable in Kotlin it will require you to correctly handle the type.

You can use the ?. syntax which will call the function when the target isn't null.
If it's null,  the call will return null instead of throwing a *NullPointerException*.

As java doesn't have support for null safety,  all java code is seen as nullable per default.

More info: [https://kotlinlang.org/docs/null-safety.html](https://kotlinlang.org/docs/null-safety.html)

## Immutability as default

Variables declared as *val* can't be changed afterwards.
You can make a variable mutable by declaring it as a *var*.
But that should only be done as an optimization.

Likewise,  all collection classes are implemented as Immutable per default,  with Mutable as an option.

Making code immutable makes it much easier to reason about,  and also makes concurrent programming easier.

## Data classes

Data classes are immutable classes.
Getters,  toString,  hashCode and Equals are automatically generated.
They also allow for destructuring.

More info:  [https://kotlinlang.org/docs/data-classes.html](https://kotlinlang.org/docs/data-classes.html)

## Sealed classes

Sealed classes are sum types in Kotlin.
They allow for having a type with a fixed set of options.

With sealed classes the compiler will also verify if you code is handling all the different options for a type.
They work great together with the pattern matching.

More info:  [https://kotlinlang.org/docs/sealed-classes.html#sealed-classes-and-when-expression](https://kotlinlang.org/docs/sealed-classes.html#sealed-classes-and-when-expression)

## (Rudimentary) pattern matching

The when expression allows a rudimentary form of pattern matching on the options of a Sealed class.
You can have different is expressions,  and it will automatically cast your variable to the type when doing so.

## Functional methods

All functional methods are implemented on the collections ( map,  flatMap,  filter,  find,  ... ).
You can also use [arrow](https://arrow-kt.io) to expand the functional types.

## Other nice things

### Operator overloading
Kotlin allows you to do operator overloading on your classes.

More info: [https://kotlinlang.org/docs/operator-overloading.html](https://kotlinlang.org/docs/operator-overloading.html)

### Scope functions

Allows you to execute blocks of code with the receiver in scope.
Read more [here](https://kotlinlang.org/docs/scope-functions.html#function-selection).

### Objects
You can create objects which are basically Singletons in Java code.

# How to learn more

- [https://kotlinlang.org](https://kotlinlang.org)
- [Functional programming with Kotlin](https://www.youtube.com/watch?v=eNe5Nokrjdg)
- [Kotlin koans](https://play.kotlinlang.org/koans/overview)