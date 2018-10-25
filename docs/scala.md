## Evaluation rules

* `def` defines a method
* `val` defines a fixed value, it is immmutable and eagerly initialized
* `var` defines a variable reference, it is mutable and it should be avoided
* `lazy` only initialised when required and as late as possible (deferred evaluation), default is strict and is not recomputed like by-name parameters

```scala
    def myFunction = 2         // evaluated when called
    val myImmutableValue = 2   // evaluated immediately
    lazy val iMLazy = 2        // evaluated once when needed
    
    def sort(x: List[Double])       // call by value
    def square(x: => List[Double])  // call by name
    // ds is a sequence of Double, containing a varying number of arguments
    def varargsFunction(ds: Double*) = ???
```

* **Call by-value**: evaluates the function arguments before calling the function
* **Call by-name**: evaluates the function first, and then evaluates the arguments if need be (each time the parameter is referenced inside the function)

## General object hierarchy:

![scala-hierarchy](assets/img/scala-hierarchy.png)

**NOTE:** All members of packages `scala` and `java.lang` as well as all members of the object `scala.Predef` are automatically imported.

* `scala.Nothing` is a trait that is the bottom subtype of every subtype of `scala.Any`
    * `scala.Any` base type of all types. It has methods `hashCode` and `toString` that can be overridden
* `scala.AnyVal` is the base type of all primitive types: `Double`, `Float`, etc.
* `scala.AnyRef` base type of all reference types. (alias of `java.lang.Object`, supertype of `java.lang.String`, `scala.List`, any user-defined class)
* `scala.Null` is a subtype of any `scala.AnyRef`, and `scala.Nothing` is a subtype of any other type without any instance.
    * `Null` is a trait and is the bottom type similiar to `Nothing` but only for `AnyRef` not `AnyVal`
    * `null` is the only instance of type `Null`
* `Nil` is an empty list that is defined as a `List[Nothing]`
* `None` is an empty option that is defined as a `Option[Nothing]`
* `Unit` is a subtype of `AnyVal`, it's only value is `()` and it is not represented by any object in the underlying runtime system. A method with return type `Unit` is analogous to a Java method which is declared `void`

## Type Parameters

Conceptually similar to C++ templates or Java generics. These can apply to classes, traits or functions.

```scala
    class TypedClass[F](arg1: F) { ??? }  
    new TypedClass[Int](1)  
    new TypedClass(1)   // the type is being inferred, i.e. determined based on the value arguments  
```
Conventionally, the type parameters are expressed using uppercase letters (e.g. *A, B, T, F*). It's also possible to restrict the type being used, e.g.

```scala
    def func[T <: TopLevel](arg: T): T = { ... } // T must derive from TopLevel or be TopLevel
    def func[T >: Level1](arg: T): T = { ... }   // T must be a supertype of Level1
    def func[T >: Level1 <: Top Level](arg: T): T = { ... }
```

## Variance

**Upper Bounds:**
`[S <: T]` means: S is a subtype of T. Let's suppose that T is actually an `Iterable`, then S could one of `Seq`, `List` or `Iterable`.

**Lower Bounds:**
`[S >: T]` means: S is a supertype of T, or T is a subtype of S. So, if T is a `List`, S could be one of `List`, `Seq`, `Iterable`, or `AnyRef`.

**Mixed Bounds:**
`[S >: T2 <: T1]` means: s is any type on interval between T1 and T2. In this case we have basically a mix of the two cases above.

Let's consider `NonEmpty <: IntSet`, then can we infer that `List[NonEmpty] <: List[IntSet]`?
Intuitively, this makes sense: a list of non-empty sets is a special case of a list of arbitrary sets.
We call types for which this relationship holds **covariant** because their subtyping relationship varies with the type parameter. Thus `Lists` in scala are **covariant**.

Does covariance make sense for all types, not just for List? No. For instance, in Scala, **arrays are not covariant**.
So when does it make sense to subtype one type with another? The answer is the **Liskov Substitution Principle**: _If `A <: B`, then everything one can to do with a value of
type B one should also be able to do with a value of type A._

Say C[T] is a parameterized type, and A, B are types such that:

* Given `A <: B` (A is a subtype of B)
* If `C[A] <: C[B]`, `C` is **covariant**
* If `C[A] >: C[B]`, `C` is **contravariant**
* Neither `C[A]` or `C[B]` is a subtype of the other, then C is **invariant** (or *"nonvariant"*). 

Scala lets you declare the variance of a type by annotating the type parameter:

```scala
    class C[+A] { ... } // C is covariant
    class C[-A] { ... } // C is contravariant
    class C[A]  { ... } // C is invariant
```

So, given that `Any` > `AnyRef` > `IntSet` > `Empty` and `NonEmpty`, if 
```scala
type A = IntSet => NonEmpty
type B = NonEmpty => IntSet
```
According to the Liskov Principle => `A <: B`, since B can return an Empty or NonEmpty, but A can return only NonEmpty.

For a function, if `A2 <: A1` and `B1 <: B2`, then `A1 => B1 <: A2 => B2`. The consequence is that functions must be **contravariant in their argument types and covariant in their result types**, e.g.

This example shows that functions are _contravariant_ in argument types and _covariant_ in return types.

```scala
package io.github.sentenza.cars

class Car {}
class SportsCar extends Car {}
class Ferrari extends SportsCar {}

object morecovariance extends App {

    // Test 1: Works as expected

    def test1( arg: SportsCar => SportsCar ) = {
        new SportsCar
    }

    def foo1(arg: Car): Ferrari = { new Ferrari }
    def foo2(arg: SportsCar): Car = { new Ferrari }
    def foo3(arg: Ferrari): Ferrari = { new Ferrari }

    test1(foo1) // compiles
    test1(foo2) // Fails due to wrong return type. 
    test1(foo3) // Fails due to wrong parameter type

}
```

Find out more about variance in [Covariance And Contravariance in Scala](http://blog.kamkor.me/Covariance-And-Contravariance-In-Scala/)

