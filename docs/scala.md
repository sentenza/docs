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
Conventionally the type parameters are expressed using uppercase letters (e.g. *A, B, T, F*).

It is possible to restrict the type being used, e.g.
```scala
    def func[T <: TopLevel](arg: T): T = { ... } // T must derive from TopLevel or be TopLevel
    def func[T >: Level1](arg: T): T = { ... }   // T must be a supertype of Level1
    def func[T >: Level1 <: Top Level](arg: T): T = { ... }
```

## Variance

* Given `A <: B`
* If `C[A] <: C[B]`, `C` is **covariant**
* If `C[A] >: C[B]`, `C` is **contravariant**

Otherwise C is **nonvariant**
```scala
    class C[+A] { ... } // C is covariant
    class C[-A] { ... } // C is contravariant
    class C[A]  { ... } // C is nonvariant
```

For a function, if `A2 <: A1` and `B1 <: B2`, then `A1 => B1 <: A2 => B2`.

Functions must be **contravariant in their argument types and covariant in their result types**, e.g.
```scala
    trait Function1[-T, +U] {
      def apply(x: T): U
    } // Variance check is OK because T is contravariant and U is covariant

    class Array[+T] {
      def update(x: T)
    } // variance checks fails
```

Find out more about variance in [Covariance And Contravariance in Scala](http://blog.kamkor.me/Covariance-And-Contravariance-In-Scala/)

