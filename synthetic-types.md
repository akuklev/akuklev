Synthetic Types
===============

The first data types one encounters in general-purpose languages are __hardware-specific types__. Well known examples from C-like languages are the data types `int` (`int32`) and `float` (`float32`) of 32-bit integers and floating point real numbers respectively. Yet, many languages also support data types defined independently of any of machine-related aspects. (TODO: сказать, что мы их будем называть __intrinsically defined types__) There are two kinds of such data types: function types and synthetic types.

Function types are defined by the way their values can be used. In particular, the type `A -> B` is defined as the type inhabited by objects that can be applied to a value of type `A` and deterministically yield a value of type `B`. Synthetic types are defined by the way their values can be constructed or synthesized, hence the name. This article deals with synthetic types, starting with most basic ones.


§ Finite synthetic types
------------------------

The most simple intrinsically defined types are finite enumerations: (TODO: think about the syntax; reader doesn't know what finite enumerations are)

**Example 1**
```c
enum State {
  Failed,
  Working
}

enum Digit {
  D0,
  D1,
  D2,
  ..,
  D9
}
```

Here, the two enumeration types, called `State` and `Digit`, are defined. `Working`, `Failed`, `D0` and so on are called the __constructors__ of respective types. The values of enumeration types can be inspected by exhaustive case analysis:
```scala
s match {
  case Failed  => do-something;
  case Working => do-something-else;
}
```
The values of intrinsically defined types have to be stored in the computer memory, and for that purpose they are mapped onto hardware-specific types. The values of enumeration types, such as the ones in the example, are normally stored as integers of sufficent bit size (`int8`, `int16`, `int32`), where each constructor is identified with a specific numerical value. In the example above, `Failed` could be assigned to 0 and `Working` to 1.

In many low-level languages including C it is possible to inspect which numeric code is used for which constructor, and to use numeric constructor codes to instantiate a variable of an enumeration type. (TODO: пример) However, such practices are error prone, moreover, they undermine the abstraction behind machine-independent types. In languages, where enumeration types are truly machine-independent, values of enumeration types can be created only by manifestly using constructors and inspected only by case analysis. It allows the compiler to chose the implementation it pressumes to be optimal on the given machine in the given setting.

Hardware-defined primitive data types such as `int32` and `float64` are enumeration types in disguise, because they can be modelled by finite sequences of bits.

§ Finite function types
-----------------------

Function types were briefly mentioned in the introduction. Let us now consider them in more detail.

For given data types `A` and `B`, the data type `A -> B` is defined as the type inhabited by values can be applied to a value of type `A` and deterministically yield a value of type `B` when applied. For this definition to be machine-independent, the functions have to be “opaque”: the values of the type `A -> B` are not allowed to be inspected in any way except by applying them to arguments of matching type. Thus, values of function types are indistinguishable iff they yield equal results for same arguments. By identifiability of indiscernables, we have: **given `f(x) = g(x)` for each `x` of type `A`, `f = g`**.

The function types `A -> B` are in general open types in the sence that type defintion does limit how values of the type `A -> B` are to be constructed: the limitation are given by the way how functions are allowed to be used and inspected. Thus on the first glance it might appear that function types are never manifestly finite.

Yet, assume both `A` and `B` are finite data types, with constructors denoted by `{A₁,.., Aₙ}` and `{B₁,..,Bₘ}`. Then, there are only `mⁿ` functions `A -> B` of the following form:
```scala
x match {
  case A₁ => B₍...₎;
  case A₂ => B₍...₎;
  ...
  case Aₙ => B₍...₎;
}
```

It can be easily shown that any functions `A -> B` is equal to one of these, thus the type `A -> B` is finite up to equality of its elements. In fact, the types `A -> B` are finite if and only if both `A` and `B` are finite.



* * *



Enumerations are finite and closed data types. Finite means that a variable of enumeration data type can attain only a finite set of values. 

Closedness means roughly that the data type does not contain anything except stuff that was explicitly put there in its definition. TODO: present nonexamples: a function type and a “enum” with function-type parameter.

Closedness means that a variable of enumeration data type is guaranteed to be given by a constructor declared in the definition of the enumeration type. In particular enumeration types cannot be extended ulteriorly and inheritance is forbidden for them.

There are several extensions to enums:
1) Constructors can be allowed to have parameters;
2) Enumeration types can be allowed to have postulated equalities.

The following example, written in a C-like pseudocode, shows both extensions:

**Example 2**
```c
enum Color {
  Red,
  Green,
  Blue,
  CustomColor(Digit r, Digit g, Digit b),
  
  // Equalities:
  Red   => CustomColor(D9, D0, D0),
  Green => CustomColor(D0, D9, D0),
  Blue  => CustomColor(D0, D0, D9)
}
```

This extensions do not spoil closedness and finiteness as long as all parameters are also given by enumerations.




§ Constructors with parameters
------------------------------



§ Closed inductive types
------------------------

One can go beyond finitness by allowing parameters of the constructors to have the type being defined. Let's consider the most basic example:

**Example 3**
```c
inductive NaturalNumber {
  Zero,
  SuccessorOf(NaturalNumber n)
}
```

The possible values of a variable of type `NaturalNumber` are `Zero`, `SuccessorOf(Zero)`, `SuccessorOf(SuccessorOf(Zero))`, etc. Such types are finite and thus not enumerations anymore. They are called closed inductive data types. Closedness refers to TODO. While not finite, closed inductive types are effectively enumberable: for each closed inductive data type one can explicitly write down a program that prints a sequence of all its possible values.

Inductive types are inherently immutable and are not allowed to contain any cycles. A number `inf = SuccessorOf(inf)` is not allowed (cycles are forbidden) and cannot be constructed because the parameters of constructors are required to be given by already defined immutable values of respective types.

Let's consider a few other examples:
**Example 4**
```c
inductive ListOfNaturals {
  EmptyList,
  NonEmptyList(NaturalNumber head, ListOfNaturals tail)
}

inductive BinaryTreeOfNaturals {
  Leaf(NaturalNumber n),
  Node(BinaryTreeOfNaturals left, BinaryTreeOfNaturals right)
}

```

Inductive types are "synthetic" in the following sense: any possible value of a variable is guaranteed to be built ("synthesised") from a fixed set of constructors 
in a non-circular fasion. In other words any value is given by a tree of constructors of finite depth. This can be also expressed as follows:
* A value of an inductive type can be thus exhaustively analysed by recursive pattern matching and the recursion is guaranteed to terminate (no cycles => no infinite loops).
* A property for all values of a given inductive type can be proven by structural induction on possible values. By the way, that is why they are called inductive.

Inductive types are also compatible with custom equalities:

**Example 5**
```c
inductive Integer {
  Positive(NaturalNumber n),
  Negative(NaturalNumber n),
  IntegerZero
  
  // Equalities:
  Positive(Zero) => IntegerZero
  Negative(Zero) => IntegerZero
}

inductive UnorderedPairOfIntegers {
  UPair(Integer a, Integer b)
  
  // Undirected equalities:
  Symmetry(Integer A, Integer B) : UPair(a, b) = UPair(b, a)
}
```

If all equalites in a closed inductive type are directed, the equality of two values is decidable, i.e. equality can be checked by an algorithm which is guaranteed to terminate. If some of the equalities ere not directed (see UnorderedPair in the Example 5), the equality checking is only guaranteed to be semidecidable, i.e. to terminate if the values are equal indeed. If the values are distinct, equality checking might run into an infinite loop. That is not a flaw of a particular equality checking algorithm, but an general problem known in mathematics as [word problem undecidability](https://en.wikipedia.org/wiki/Word_problem_(mathematics)).

§ More General Closed Synthetic Types
-------------------------------------

There are several important extensions to inductive types.

First of all, one may allow to define a family mutually dependent inductive types at once. It may be a finite number of types:

**Example 6**
```c
inductive VariableWidthTreeOfInts, ListOfTrees {
  Leaf(Integer n) : VariableWidthTreeOfInts,
  Node(ListOfTrees list) : VariableWidthTreeOfInts,
  EmptyList : ListOfTrees,
  NonEmptyList(VariableWidthTreeOfInts head, ListOfTrees tail)
}
```

It also may be an infinite family of types indexed by a parameter, in which case one also has to allow dependent signagures for constructors:

**Example 7**
```c
inductive LengthIndexedListOfIntegers(NaturalNumber length) {
   EmptyList : LengthIndexedListOfIntegers(Zero),
   NonEmptyList(Integer head, 
                NaturalNumber tail_length,
                LengthIndexedListOfIntegers(tail_length) tail
     ) : LengthIndexedListOfIntegers(tail_length + 1)
}
```

Here, the type of the index `NaturalNumber` is defined in advance, but there is no problem in defining one or more index types simultaneously with everything else. This extension is known as inductive-inductive types. The example 7 also uses an operation (`+ 1`) defined on the index type `NaturalNumber`, which was also defined in advance. When the index type is being defined simultaneously with other types, all required operations on this type have to be also defined simultanously. This extension is known as small inductive-inductive-recursive types.

With all these extensions, one can define very complex data types such as 
* intricataly balanced trees representing internal states of advanced for data structures (such as Red-Black trees, B* trees, etc.),
* abstract syntax trees for languages, including languages with types and binders¹.

Notwithstanding all this extensions, the inductive types retain their basic properties:
* Synthetic, i.e. built from fixed set of constructors and can be analysed by top-down-recursive pattern matching;
* Closed, if all paremeters of all constructors are themselves closed:
  * Effectively enumerable;
  * Have semidecidable equality;

----
1. Closed synthetic types provide syntactic models (= initial models) for all finitary extended algebraic theories, in particular for all generalized algebraic theories without sort equations. Non-closed synthetic types (see below) are capable of dealing with countably infinitary extended algebraic theories.

§ Non-Closed Synthetic Types
----------------------------

There are data types which cannot be expressed by closed synthetic types.  

The most prominent examples are given by functions on infinite data types (for instance infinite sequences of digits `NaturalNumber -> Digit`) and exact real numbers `Real`. While this two types are perfectly valid data types, it is a matter of argument if infinite sequences and exact real numbers are really “data” because they cannot be stored on a finite digital carrier or sent over network in a finite amount of time.

Yet it is possible to define the types of __computable__ sequences and __computable__ real numbers. That is, ones sequences that can be produced by an algorithm and real numbers that can be algorithmically computed to any desired finite precision. These are unequivocally **data** types, because they can be easily sent over network or stored in form of the respective lambda expression (or any other desired form of Turing-complete computation). These two types still cannot be represented by closed synthetic types, because closed synthetic types are by design effectively enumerable and computable sequences/computable reals are not effectively enumerable due to [halting problem](https://en.wikipedia.org/wiki/Halting_problem).

The type of all lambda expressions can be certainly given by as a closed synthetic type, but it will neccesarily include some nonterminating expressions (the ones that does not correspond to any valid sequence and no valid real number respectively), and have the wrong notion of equality. As we already mentioned, equality on closes synthetic types is semidecidable in positive sense (guaranteed to terminate if the values are equal), while equality of reals and equality of discernable sequences is semidecidable in the negative sense: one can check equality of two numbers digit-by-digit (equality of sequences respectively item-by-item) and this process is guaranteed to terminate if there is a difference somewhere, but would last infinitely when values are indeed equal.

To have a mathematically sound type system, one has to accept the existence of open types of functions `A -> B` (where `A` is an infinite type) which are inhabited by constructively definable functions from `A` to `B`, but not limited to them. The only thing one can be sure, is that such a function can be applied to a value of type `A` and yields a value of type `B`, otherwise an kind of “open-world assumption” applies.

Non-closed synthetic types are just like synthetic types, but the constructors are allowed to have parameters of open types (primarily `A -> B`). The types of exact reals `Real` and partial computations `℧(T)` can be expressed as non-closed synthetic types employing parameters of the kind `A -> B`. This also allows to define such complicated mathematicians' types as computable reals, computable functions `Comp(A, B)`, continious functions `𝒞(A, B)`, borel functions `𝔅(A, B)` measurable functions `Meas(A, B)`, etc.

Non-closed types are not effectively enumerable anymore. If non-closedness arises form constructor parameters of types `A -> B` where `A` infinite, they are not positively semidecidable anymore. They might be discernable (i.e. have negatively semidecidable equality) or entirely undecidable.

Non-closed synthetic types provide initial models for contably-infinitary algebraic theories (like the theory of compact Hausdorff spaces), but these cannot be called syntactic models anymore.

Another kind of non-closedness arises when the functions defined simultaneously with their domain in inductive-inductive types are allowed to be kind-valued. Such extention is known as fibered inductive types (or large inductive-inductive-recursion). Since kinds are inherently open virtual types, resulting synthetic types are non-closed as well. This extension allows defining universes and abstract syntax trees for languages with extensible type systems. In particular, such types are capable of capturing abstract syntax of their host language, which is known as “language eating itself”. In the latter case, types remain positively semidecidable and effectively enumerable relative to the host language, which allows calling them syntactic models, at least in relativised sense.
