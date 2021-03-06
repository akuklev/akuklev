Declarative Approach to Data Types
==================================

I work in [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). This article is an introduction to declarative data type definitions for interested software engineers, computer scientists, and mathematicians willing to tolerate programming-centered perspective. At the same time, this article contains novel material based on mathematical results by the members of our group which are yet to be published.

Declaratively defined data types are data types specified in terms of _what_ they are good for, rather than _how_ they are implemented. Declarative data type definitions are essential for abstract reasoning about programs. Such definitions also form the frame for the emerging structuralist foundation of mathematics[[1]](https://arxiv.org/abs/2009.09541).

It is importaint to note, that throughout this article series, the term _“data types”_ will be used in the following narrow sense: while _types_ in general can refer to objects (such as files and mutable data structures), data types refer solely to _data_, by which we mean self-conatined indefinitely copyable pieces of information like values of variables or content of files at some point time. _Object types_ are beyond scope of this article. 

We believe that the best way to approach to this hefty topic is by considering examples of increasing complexity. Since no mainstream programming language supports declarative data type definitions in sufficient generality, a pseudocode will be used in all examples. We will start with the definitions of finite types and describe how to declaratively construct primitive numeric data types such as the type `int32` of 32 bit integers. Then we will define the types for unbounded natural, integer, and rational numbers. After that we will make a digression to define container types (such as pairs, lists, binary trees, etc.) and complex trees.

**Table of contents**
* [**Defining finite types:** Variant data types]()
* [**Recovering primitive types:** Variant Types with bundled operations]()
* [**Beyond finite types:** Inductive types]()
* [**Defining integers:** Inductive Types with reduction rules]()
* [**Defining rationals:** Quotient Inductive Types]()
* [**Defining containers:** Polymorphic Inductive Types]()
* [**Defining syntax trees:** Dependent Inductive Types]()
* [**Towards reals:** Beyond Synthetic Types]()

§ Two Basic Approaches
----------------------

We will be considering two approaches to declarative type definitions:
* Black box approach
* Bottom-up approach

Let us illustrate the approaches by examples. Assume, the type `Bit` a finite type with exactly two values `tt` (true), and `ff` (false) and consider the following two data type defintions:

**Example 1**
```scala
structure BitSequence {
  head : Bit
  tail : BitSequence
}

construct BitList {
   EmptyBitList,
   NonEmptyBitList(head : Bit, tail : BitList)
}
```

The first definition is a black box definition. It says, a piece of data is a value of type `BitSequence` whenever it has a `head` of type `Bit` and a `tail` of type `BitSequence`. The definition does not make any assumptions how the values are built.

The second definition is a bottom-up definition. It says tha

Both limit value space: black-box can be only observed by defined operators, no way to distinguish observationally equivalent values. bottom-up: one can safely assume that any value es equal to one built from constructors.

Bottom-up types: number of possible values exactly known. even if infinite, all possible values can be systematically enumerated exhaustively.
Black-box types are open. We have a lower bound (there is at least a countably infinite number of bit sequences, namely constructible bit sequences) and sometimes an upper bound in terms of other blackbox types (“number” of bit sequences is bounded from above by the “number” of subsets of natural numbers).


§ Defining finite types: Variant data types
-------------------------------------------

We shall begin with the simplest kind of types: the ones defined by a finite enumeration of their possible values. These types are called _variant types_. Here is an example:

**Example 1**
```scala
datatype BasicColor {
  Red
  Green
  Blue
  Gray(intensity : 0..100)
}
```

Here, `BasicColor` is defined as a type with allowed values `Red`, `Green`, `Blue`, and `Gray` of specific `intensity` given by an integer between 0 and 100. `Red`, `Green`, `Blue`, and `Gray` are called the _constructors_ of the type `BasicColor`. The constructor `Gray` is said to be a _parametrized_ constructor, while the other three are called _atomic_. The values of variant types can be inspected by exhaustive case analysis:
```scala
s match {
  case Red   => ...
  case Green => ...
  case Blue  => ...
  case Gray(intensity) => ...
}
```

**Definition 1**
> _Variant data types_ are data types specified by a finite list of named constructors. Constructors can either be atomic or have a finite number of parameters of previously defined types. Values of variant datatypes can be created only by manifestly using a constructor from the list. Variables of variant data types can be inspected only by case analysis. These are the only permissible interactions with variables of variant data types.

Variant data types without parametized constructors are finite, i.e. variables of the respective types can attain only a finite number of values. This also applies to variant types with parametrized constructors as long as all parameters are of finite types.

It may happen that multiple variant types have identically named constructors. For disambiguation, qualified names such as `BasicColor.Red` will be used in our pseudocode.

The values of declarative data types have to be stored in the computer memory, and for that purpose they are mapped onto hardware-specific data structures. For instance, finite variant types can be stored as integers of sufficent bit size (`int8`, `int16`, `int32`), where each constructor is identified with a specific numerical value. In the example above, `BasicColor` could be stored as an `int8`, where `Red` could be assigned to -1 and `Green` to -2, `Blue` to -3, and the remaining 100 shades of gray to the numbers 0 to 100. Alternatively, one could have chosen any other numerical codes or used a varable length encoding: four bits for the constructor variant, and additional 7 bits for the `intensity` field if the constructor happens to be `Gray`. All declaratively defined data types have multiple implementations, none of which is inherently universally superior. For example, compact variable length encodings are often better for transporting data over the network, whereas fixed length encodings perform better in RAM. When working with declaratively defined data types, the compiler can chose the implementation it pressumes to be optimal on the given machine in the given setting, or even switch implementations depending on medium. For all declarative types, the compiler will be able to generate at least one default implementation. However, the automatically generated implementation is often suboptimal. Thus, the oportunity to provide custom alternative implementations that the compiler can chose from is a desirable feature. The program cannot inspect which implementation is used as the implementation is unspecified by the type definition. Furthermore, definition 1 explicitly forbids any interactions within the program that could possibly reveal the implementation. This feature is known as implementation opaqueness.

§ Defining primitive types declaratively: Variant Types with bundled operations
-------------------------------------------------------------------------------

On the hardware level, computers support basic logical and arithmetic operations on a number of types, such as `boolean`, `int32` of 32 bit integers, `float64` of 64-bit floating point approximations of real numbers. Such types are provided as inbuilt data types in most programming languages, and called primitive numeric data types. Internally, values of those types are just finite-length bit sequences and thus can be modelled by variant types. In order for operations on such models to be able to interact with native implementations, one needs to introduce reducible constructors.

In the examples above, distinct constructors always referred to distinct values. Variant data types can be extended to contain constructors that refer to values that are readily expressible by other constructors. Such constructors are called _reducible_.

**Example 2**
```scala
datatype BasicColor {
  Red
  Green
  Blue
  Gray(intensity : 0..100)
  Black => Gray(100)
  White => Gray(0)
}
```

Here, `Black` and `White` are reducible constructors and all other constructors are called _irreducible_.

For reducible constructors, the restriction that parameters must be of previously types can be lifted. One can also allow parameters of type being the type being defined, without rendering the type infinite.(TODO: ещё раз подумать над формулировкой) Consider the following example:

**Example 2**
```scala
datatype Bool {
  True
  False
  Not(a : Bool)
  And(a : Bool, b : Bool)
  
  Not(True) => False
  Not(False) => True
  
  And(True, True) => True
  And(True, False) => False
  And(False, True) => False
  And(False, False) => False
  
  ... // all other logical operations 
}
```

This example uses reducible constructors `Not(x)` and `And(x, y)` to define operations on the type `Bool` while defining the type itself. An operation on the type defined using reducible constructors is called a _bundled_ operation. When a custom implementation for a type is provided, bundled operations can be provided efficient custom implementations too.

In the example above, bundled operations have already allowed us to declaratively define the primitive data type `Bool` together with all hardware operations on it. Similarily we can define the type `Int8` of 8-bit integers:

**Example 3**
```scala
datatype Int8 {
  Int8(b1 : Bool, b2 : Bool,.., b8 : Bool)
  
  // Bitwise logical operations:
  BitwiseNot(x : Int8)
  BitwiseAnd(a : Int8, b : Int8)
  BitwiseXor(a : Int8, b : Int8)
  
  // Arithmetic operations:
  Negated(a : Int8)
  Sum(a : Int8, b : Int8)
  Product(a : Int8, b : Int8)
  Quotient(a : Int8, b : Int8)
  Remainder(a : Int8, b : Int8)
  
  BitwiseAnd(Int8(a1,.., a8), Int8(b1,..,b8)) => Int8(And(a1, b1), And(a2, b2),.., And(a8, b8))
  BitwiseXor(Int8(a1,.., a8), Int8(b1,..,b8)) => Int8(Xor(a1, b1), Xor(a2, b2),.., Xor(a8, b8))
  
  // now all arithmetical operations can be defined in terms of bitwise ones...
}
```

All hardware-defined primitive data types can be modeled by finite variant types declaratively using bit sequences. The native implementations of those data types and operations on them can be regarded as predifined “custom implementations”.

Reducible constructors and bundled operations in particular play an important role in context of generic programming, which will be discussed in section [TODO].


§ Beyond finite types: Inductive types
--------------------------------------

The fact that all low level primitive datatypes like `int32` are finite types, can give a misleading impression that all data types are finite. That, however, is not true under the assumption of potentially infinite memory, i.e. the assumption that whenever a program runs out of memory and stalls, one can always expand data storage and resume the program execution. The prime examples of infinite data types are the types of (unbounded) natural and integer numbers.

To construct the types of natural and integer numbers declaratively, one needs an extension to the concept of variant types: the _inductive types_. In their simplest form, inductive types are recursive “variant types”, i.e. the __parameters__ of their constructors are allowed to be of the type being defined:

**Example 4**
```scala
datatype Nat {
  Zero
  Succ(pred : Nat)   // successor of the natural number `pred`
}
```

The possible values of the type `Nat` are thus `Zero`, `Succ(Zero)`, `Succ(Succ(Zero))`, etc. This definition does not allow for cyclic values. In particular, there can be no such `n : Nat` that `n = Succ(n)`.

In further examples numeric literals like `124` will be regarded as values of the infinite type `Nat`, namely as shorthands for `Succ(...Succ(Zero))` with the respective number of `Succ` constructors, rather than values of a primitive data type like `int32`.

As with variant types, the values of inductive types can be only inspected by exhaustive case analysis. However, for inductive types exhaustive case analysis supports well-founded recursion. In order to define a function `f` on `Nat`, one has to define it on `Zero` and on `Succ(n)` under assumption that `f(n)` is already known:

**Example 5**
```scala
def isEven(n : Nat) : Boolean
  Zero => True
  Succ(pred) => Not(isEven(pred))
```

Acircularity of inductive types amounts to the property that such functions always terminate, i.e. cannot fall into an endless loop.

The name “inductive types” comes from the mathematical concept of inductive proof. A proof by induction is a proof that a proposition `P(n)` holds for all natural numbers `n : Nat` by demonstrating that it holds for `Zero` and holds for `Succ(n)` whenever it is already established for `n`.



§ Defining integers: Inductive Types with reduction rules
---------------------------------------------------------

Inductive types may have recursive reducible constructors. These generalize reducible constructors of variant types, that were introduced in [#Variant_Types_with_bundled_operations]. Let us define some bundled operations for `Nat` to provide a solid example:

**Example 6**
```scala
datatype Nat {
  Zero
  Succ(pred : Nat)
  
  Sum(n : Nat, m : Nat)
  Prod(n : Nat, m : Nat)
  
  Sum(n, Zero) => n
  Sum(n, Succ(m)) => Succ(Add(n, m))
  
  Prod(n, Zero) => Zero
  Prod(n, Succ(m)) => Sum(n, Prod(n, m))
}
```

Partially reducible constructors, are constructors for which reductions are defined by non-exhaustive case analysis:

**Example 7**
```scala
datatype Int {
  FromNat(n : Nat)
  Negated(n : Nat)
  Zero,
  
  FromNat(Zero) => Zero
  Negated(Zero) => Zero
}
```

**Technical remark:** From now on, we'll be using notation `+n` for `FromNat(n)` and `-n` for `Negated(n)`.

When performing exhausitve case analysis, reducible cases do not appear. Let us define negation for integers to illustrate:

**Example 8**
```scala
def Incremented(z : Int) : Int
  Zero => FromNat(Succ(Zero))
  Negated(Succ(n)) => Negated(n)
  FromNat(Succ(n)) => FromNat(Succ(Succ(n)))
}
```
Here, the cases `FromNat(Zero)` and `Negated(Zero)` are not mentioned at all because `Incremented` _has_ to reduce to the same value as `Incremented(Zero)` in this case.

Note, that the decision which constructors are reducible is non-unique. For example, one could have also defined integers as follows:
**Example 9**
```scala
datatype Int {
  Pos(n : Nat.Succ)
  Neg(n : Nat.Succ)
  Zero
  
  FromNat(n : Nat)
  
  FromNat(Zero) => Zero
  FromNat(Succ(n)) => Pos(Succ(n))
}
```

For each inductive type `T` with reducible constructors, one could define a type of “raw terms” `RawT`: the type with exactly the same constructors but without any reduction rules. Any function `f` on the original type `T` is also a function on the type of raw terms `RawT` under the hood, yet with a property that on values `x : RawT` and `y : RawT` which reduce to the same value if seen as `T`-value, `f` yields _literally_ the same results. This property is guaranteed by opaqueness of declarative types, which means that functions one defines are only allowed to inspect values of inductive types by exhaustive case analysis, and the way exhaustive case analysis works for reducible constructors (see example 8).

<details><summary><b>Note for experts</b></summary>
<p>
Availability of partially reducible constructors substantially increase the strength of inductive types. They allow to enforce _literal_ equalities on functions from a given type without introducing strict equality types into the type system. In particular, they render semi-simplicial types definable in a univalent type theory.
</p>
</details>


§ Defining rationals: Quotient Inductive Types
----------------------------------------------

Let us briefly mention another feature that enables definition of rational numbers.

Rational numbers are fractions with integer numerator and denominator, and their the usual definition of rational numbers goes as follows:
> Rational numbers are pairs of an integer called numerator and a strictly-positive integer called denumerator, up to identification of pairs `(num, den)` and `(num · q, den · q)` where `q` is a positive integer.

It turns out to be possible to allow custom identifications in inductive types. Without going any deeper into the matter right now, let us just write down a definition:

**Example 9**
```
datatype Rational {
  Fraction(num : Int, den : Int.Pos)
  
  ReduceFraction(num : Int, den : Int.Pos, q : Int.Pos) : 
    Frac(Prod(num, q), Prod(den, q)) = Frac(num, den)
}
``` 

§ Defining containers: Polymorphic Inductive Types
--------------------------------------------------

With inductive types we can also container types such as lists:

**Example 6**
```
datatype ListOfNats {
  EmptyList,
  NonEmptyList(head : Nat, tail : ListOfNats)
}
```

It is of course desirable to have a generic defintion of lists regardless of their element type. For this occasion, types are allowed to have parameters themselves. This feature is known as parametric polymorphism.

**Example 7**
```
datatype List<T : *> {
  EmptyList,
  NonEmptyList(head : T, tail : List<T>)
}
```

Here the parameter `T` if of type “data type” which is written as `*`. In the type specification the parameter `T` is used as type for the first parameter `head` of the `NonEmptyList` constructor and as a parameter of the type `List<T>` for its second parameter `tail`. The type `*` itself is not a _data_ type, it belongs to another class of types known as virtual types: these are the types of so called compile time parameters, that never appear in the run-time. 

**Technical remark:** Constructors of different specializations of generic types are distinct constructors. In rare cases when disambiguation is necessary, their qualified names look like `List<Nat>.EmptyList` or `List<Int>.NonEmptyList`. Shorthands like `NonEmptyList<Int>` are also possible.

Types may have parameters of other types as well. For example one can define the type `Array<Type : *, length : Nat>` of lists of predefined length. However we'll postpone the definition to the section [Advanced inductive types].

* * *

By combining parametric polymorphism with custom identifications one can define such containers as unordered pairs, cycles (“linked lists without distinguised starting point”), collections (“lists up to permutations”), etc.

**Example 10**
```
datatype UnorderedPairOfInts {
  UPair(a : Int, b : Int)
  
  Swap(a : Int, b : Int) :
    UPair(a, b) = UPair(b, a)
}
```


<details><summary><b>Note for experts</b></summary>
<p>
It is tempting to think that the right way to define a symmetric binary relation on a type `T` is to define it on `UPair<T>`. It turns out that this approach unfairly trivializes the case when both elements of the pair are the same. The right way to deal with symmetric binary relations is to use the higher inductive type

```scala
higher-datatype UnorderedPairOfInts {
  HUPair(a : Int, b : Int)
  
  Swap(a : Int, b : Int) :
    UPair(a, b) = UPair(b, a)
}
```
which does not assume that for `loop(a) := Swap(a, b)` the equation `loop • loop = loop` holds. That's precisely the point where homotopic considerations arise in the type theory.
</p>
</details>

§ Defining Syntax Trees: Inductive-inductive Types
--------------------------------------------------

Inductive types can be combined with dependent signatures, yielding so called inductive type families. This can be pushed even further by allowing to define the (synthetic) type the family is indexed by simultaneously with indictive family itself. Such types are called inductive-inductive types. Quotient inductive-inductive types are excelent tool for defining types of abstract syntax trees for various formalized languages. In fact, syntactic model of every finitary generalized algebraic theory can be described in this way. It goes even beyond this: in terms of inductive-inductive types with reduction rules it is possible to describe syntactic models of all finitary extended algebraic theories and thus capture the language of type theories within type theories. 

**TODO:** Example, for instance calculator language
```
datatype VarSelector {
    Empty : VarSelector
    Left(tail : VarSelector) : VarSelector
    Right(tail : VarSelector) : VarSelector
    Both(tail : VarSelector) : VarSelector
    
    CountLeft(v : VarSelector) : Nat
    CountLeft(Empty) => 0
    CountLeft(Left(tail)) => Succ(CountLeft(tail))
    CountLeft(Both(tail)) => Succ(CountLeft(tail))
    CountLeft(Right(tail)) => CountLeft(tail)
    
    CountRight(v : VarSelector) : Nat
    CountRight(Empty) => 0
    CountRight(Left(tail)) => CountRight(tail)
    CountRight(Both(tail)) => Succ(CountRight(tail))
    CountLeft(Right(tail)) => Succ(CountRight(tail))

    Count(...)
}

datatype Expr(nVars : Nat) {
   Constant(z : Int) : Expr(Zeor)
   Vor : Expr(1)
   Neg[n](a : Expr[n]) : Expr(n)
   Add(s : VarSelector, a : Expr(CountLeft(s)), b : CountRight(s)) : Expr(Count(s))
   Mul(s : VarSelector, a : Expr(CountLeft(s)), b : CountRight(s)) : Expr(Count(s))
}
```

§ Synthetic and Behaviorial definitions
---------------------------------------

The types discussed so far were all synthetically defined, but it turns out that synthetic defintions are not sufficient for some needs. In particular, synthetic types are not sufficient to accomodate potentially unbounded containers such as (infinite) sequences `Seq<T>`, infinite trees and most fundamentally, function types `A -> B`, where `A` is an infinite type. Real numbers also cannot be expressed as purely synthetic types.

In general, there are two approaches to declarative type definitions:
* Synthetic (or closed) paradigm, where types are specified in terms of how their values are formed bottom-up.
* Behaviorial (or open) paradigm, where types are specified in terms of operations that can be applied to values of that type, without explicitly limiting how values are constructed.

```scala
structure SequenceOfNats {
   head : Nat
   tail : SequenceOfNats
}
```

Here we define the type `SequenceOfNats` with two operations that can be applied to a value of that type:
* `head` extracts the first element of the sequence;
* `tail` extracts the sequence of all elements but the first.

Allowed values of such type are “black boxes” supporting those two operations, there are no assumptions about how they are built. This approach is also known as duck typing: "if it walks like a duck and it quacks like a duck, then it's duck".


Types defined using the synthetic paradigm are closed in the sense they do not contain anything one has not explicitly put there. The number of possible values of a synthetic type is known upfront. If it is infinite, than it has to be countably infinite, i.e. all possible values can be enumerated. By contrast, for behaviorially defined types one does not assume that they are exhausted by values known upfront. , so in general one only knows a lower bound on their size. (TODO: неочевидно, почему это вообще так)





Recall that variant types are finite if they have no parametric constructors or if all they their constructors only have parameters of finite types. One can formulate a similar statement for inductive types. For inductive datatypes, it makes sense to distinguiss between parametrized constructors containing only the parameters of type being defined (endogenic constructors, like `Succ(n : Nat)`) and the ones with parameters of foreign types (exogenic constructors). Atomic constructors should be considered endogenic as well.

**Definition**
> Inductive types are called synthetic if they have no exogenic constructors or if their exogenic constructors have only synthetic parameters.

Synthetic types are particularily well-behaved: each possible value of a synthetic type can be given as a finite tree of constructors. In particular, it means that synthetic types are effectively enumerable: one can write down an algorithm that prints out possible values of a given type one-by-one and will eventually print out each of them.

Synthetic types that do not employ user-defined identifications, equality of values is decidable, i.e. equality can be checked by an algorithm which is guaranteed to terminate. For synthetic types employing postulated identifications, the equality checking is only guaranteed to be verifiable, i.e. it is guaranteed to terminate if the values are indeed equal, but might turn out to run indefinitely if the values are distinct. That is not a flaw of a particular equality checking algorithm, but an general problem known in mathematics as [word problem undecidability](https://en.wikipedia.org/wiki/Word_problem_(mathematics)).


Now consider the type of computable bit sequences `CSeq<Bool>`. It is a valid data type beyond any doubt. While being infinite sequences, its values can even be sent over the network as finite pieces of data: the computability requirement means that every such sequence can be represented as a Turing machine that produces its bits one-by-one. This Turing machine can be encoded as a finite piece of data and sent over the network.

Now let us assume this type can be specified as a synthetic data type. Than it would have verifiable equality. But that cannot be the case for sequences, even for computable ones. Their equality is of precisely opposite nature: it is not verifiable, but refutable. In order to check if two bit sequences are equal one has to compare their bits one by one. This process is guaranteed to terminate if there is a difference at some point, but would run indefinitely if the sequences are equal. Equality of sequences (no matter if computable or unrestricted) being not verifiable contradicts the assumption that their type can be specified in a synthetic fashion.

§ Structure types 
-----------------

The type of sequences (unrestricted ones this time) can be given the following behaviorial definition:

**Example 11**
```scala
structure Seq<T : *> {
  head : T
  tail : Seq<T>
}
```

This definition says, a sequence `Seq<T>` is anything for which two operations are defined: `head` yielding a value of type `T`, and `tail` yielding a value of `Seq<T>`. Since _data_ types deal only with data (indefinitely copyable pieces of information), it both operations are implicitly guaranteed to be deterministic and side-effect free.

**TODO:** Say a word on structures on types, such as order or ring structure

Mixed paradigm types can be constructed as inductive types with exogenic constructors having non-synthetic parameters:

**Example 12**
```scala
datatype InfinitelyBranchingTree<T : *>:
  Leaf(value : T)
  Node(branches : Seq<InfinitelyBranchingTree<T>>)
```

Such types as real numbers, computable real numbers, computable sequences, etc. are defined as quotient inductive types with exogenic constructors.

Non-expressability of those types by synthetic types only, means that by the very nature of things, one has to accept the existence of open types like `Seq<Bool>` that are inhabited by constructively definable sequences, but not limited to them. Cardinality of the type `Seq<Bool>` has a lower bound but not a specific value: within constructive setting the Cantorian diagonal argument does not imply that cardinalty of `Seq<Bool>` is strictly larger than countable, but rather that the assumption that one can systematically list all inhabitants of `Seq<Bool>` is contradictory. In the constructive setting Cantorian argument states that types like `Seq<Bool>` inherently satisfy an open-world assumption.

<details><summary><b>Note for experts</b></summary>
<p>
As it was mentioned, in the constructive setting cardinality of `Seq<Bool>` does not have a specific value but only a lower bound of being countably infinite. It is not a weakness of constructive setting but its strength, because it precisely reflects the model-theoretic state of affairs: in natural set-theoretic models the type `Seq<Bool>` is modelled by an uncountably infinite set with a cardinality of continuum, whereas in natural computational models the type `Seq<Bool>` is modeled by the countably infinite set of computable bit sequences.
</p>
</details>


§ Function types
----------------

For given data types `X` and `Y`, the data type of functions `X -> Y` is defined as the type inhabited by values can be applied to a value of type `X` and deterministically yield a value of type `Y`. Normally, application of function `f : A -> B` is written `f(x)`. But we can write down a verbose definition using the same syntax we used to define the type `Seq`:

**Example 13
```scala
structure Function<X : *, Y : Y> {
  apply(x : X) : Y
}
```

To apply `f : Function<X, Y>` one would have to write `f.apply(x)`.

For such definition to be machine-independent, functions have to be “opaque”: the values of the type `A -> B` are not allowed to be inspected in any way except by applying them to arguments of matching type. Thus, values of function types are indistinguishable if they yield equal results for same arguments. By identifiability of indiscernables (a principle which holds in univalent construction calculi), we have: **given** `f(x) = g(x)` **for each** `x` **of type** `A`, `f = g`.

The function types `A -> B` are in general open types in the sence that type defintion does limit how values of the type `A -> B` are to be constructed: the limitation are only given by the way how functions are allowed to be used and inspected. Thus on the first glance it might appear that function types are never manifestly finite.

Yet, assume both `A` and `B` are finite data types, with constructors denoted by `{A₁,.., Aₙ}` and `{B₁,..,Bₘ}`. Then, there are only `mⁿ` functions `A -> B` of the following form:
```scala
x match {
  case A₁ => B₍...₎;
  case A₂ => B₍...₎;
  ...
  case Aₙ => B₍...₎;
}
```

It can be easily shown that any functions `A -> B` is equal to one of these, thus the type `A -> B` is finite up to equality of its elements. In fact, the types `A -> B` are finite if and only if both `A` and `B` are finite. Along the same line of reasoning one can show that the type `A -> B` is synthetic if and only if `A` is finite and `B` synthetic.

§ Universes
-----------

Reification principle:

The type of all data types * is not a data type itself, it's a virtual type. But it can be approximated by data types:  
At any point we can define the data type of all data types defined _so far_ (including being closed with regards to all parametrically polymorphic types like `List<T : *>` and `Function<X : *, Y : 0X -> *>` defined _so far_) with identifications between types given by equivalences. Such type is called a univalent (Mahlo) universe. It does not contain itself, but the universe defined right in the next line would by definition contain all the types defined _so far_, thus in particular the previous universe. There seems to be no problem with the very first universe containing all synthetic types and all propositions, but universes themselves are _never_ synthetic. Each universe `U` comes with “decoding operator” type family `<typeTag : U> : *`. Safe large elimination is facilitated by first eliminating into a universe, and than applying `<_>`. Safety (paradox avoidance) is safeguarded by the property that the universes only contain the types defined before them, so one cannot produce vicious circles.

(Clarification: Cedillian polymorphic types like `(T : *) -> T -> T` and `* -> *` are certainly open and large, i.e. do not belong to any universe, unless relativised to one. In the same time, Cedillian "inductive types", while appearing large, are synthetic types in disguise. Thus can be assumed to belong already to the very first universe. Can be checked to following way: take a universe, create an internal universe inside, relativise the type inside internal universe, show equivalence. If resizing into smaller universe is possible, than its non-contradictory to postulate this stuff exists in all universes.)

Reflection principle:

Assume, U is a universe. Any function `0U -> T` can be lifted to `* -> T`. In particular, if we prove anything for parametrically polymorphic type relativised to a fixed universe like `Category<Ob : 0U, Hom : Ob -> Ob -> 0U> : U'`, we automatically also prove it for respective generic parametrically polymorphic type, like `Category : (Ob : *) -> (Hom : Ob -> Ob -> Ob) -> *`

TODO: Tell how reification + reflection give transport between equivalent types and paremetricity.

Let us call the type theory based on declaratively defined datatypes as presented above, virtual types reflecting parametrically polymorphic definitions, reification principle and reflection principle Univalent Calculus of Constructions. We conjecture that this theory can be seen as conservative extension of the [ZMC/S](https://golem.ph.utexas.edu/category/2009/11/feferman_set_theory.html) set theory and thus equiconsistent with ZMC.

---

### Parametricity

Take
```
f : (T : *) -> T -> T
```

We can relativise `f` to a universe `U`, yielding
```
f^U : (T : U) -> <T> -> <T>`
```

Every function does that:
```
f (e : A = B) (e' : x =[e]= y) : f(x) =[e]= f(y)
```

Parametricity is a combination of application to heteroequaluty and relativization:
```
f (r : A ⋈ B) (r' : x =[r]= y) : f^r(0)(x) =[e]= f^r(1)(y)
```

§ Very dependent types
----------------------


Very dependent functions on Nat

```
datatype DepTypeList[n : Nat]:
  Zero => Unit
  Succ(n) => Σ(T : DepTypeList[n]), (T -> U)

structure DepTypeSeq:
  initialSegment(n : Nat) : DepTypeList[n : Nat]
  
  initialSegment(Succ(n)).fst <= initialSegment(n)

// (T' : (T : *) x (T -> *)) x (T' -> *) ...
// This guys are called semi-simplicial types

DepArray[length : Nat, T : DepTypeList[length]]:
  length = 0 => EmptyArray
  length = Succ(n), d : DepTypeList[n + 1] =>
    Snoc(tail : DepArray[n, d.fst], head : d.snd(tail))
    
DepSeq[T : DepTypeSeq]:
  initialSegment(n : Nat) : DepArray[n, T.initialSegment(n)]
  
  initialSegment(Succ(n)).tail < initialSegment(n)
```


Расширяемые сорта — параметры, могут быть индексированы другими параметрами и кофибрантными сортами
Кофибрантные сорта — внутренние синтетические типы, могут быть зависимы от параметров
```
MultiCat[Ob : *, Hom : List[Ob] -> Ob -> *] where
   EmptyList : List[Ob]
   Cons...
   Cont

—
struct Pair[X : *, Y : X -> *]:
   fst : X
   snd : Y(fst) 


T(n.right) = T(n.left.head)

—
Тип DepTypeSeq определён на 
Zero
Succ(n)
Succ(n).left -> n

Zero => Unit
Succ(n) =>
```
