Declarative Data Types
======================

I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). This article is introduction to declarative data types for an interested reader familiar with some class-based programming language like C++, C# or Java.

_**Disclaimer:** Throughout this article series, the term “data types” will be used in the narrow sense. While types in general can refer to objects such as files and mutable data structures, data types refer to data, by which we mean self-conatined indefinitely copyable pieces of information like values of variables or content of files at at a given point in time. Object types are beyond scope of this article._

General purpose data types:
* Finite data types: a binary flag (yes/no), a ternary flag (pos/neutral/neg), a fixed range of integers, an alphabetic character;
* Numeric data types: a natural, integer, rational, or real number;
* Immutable container data types: a pair, a list, a binary tree or variadic tree of values;
* Potentially infinite containers: sequences, functions, potentially infinite trees.

Domain-specific data types:
* Abstract syntax trees for various formalized languages;
* Types for specific types of graphs (ordered, unordered, weighted, multigraphs, hypergraphs, etc.), networks, automata, combinatorial configuration spaces, etc.

Declarative data types are user-defined data types specified in terms of _what_ they are rather than _how_ they are implemented. Working with declarative data types allows to conceptualize the problem domain and to reason about programs in a completely architecture independent fashion.


There two approaches to declarative definitions 
* Synthetic approach (closed): Bottom-up
* Behaviorial approach (open): 
* Mixed: neccessary to define real numbers.

We are going to start with basic examples of declarative types and work our way to the most advanced ones. Since no mainstream programming language supports declarative datatypes in sufficient generality, pseudocode will be used in all examples.

§ Variant data types
--------------------

Variant types are a particular kind of user-defined data types. Let us begin with an example: `BasicColor` is defined as a type with allowed values `Red`, `Green`, `Blue`, and `Gray` of specific `intensity` given by an integer number between 0 and 100.

**Example 1**
```

datatype BasicColor {
  Red,
  Green,
  Blue,
  Gray(intensity : 0..100)
}
```

`Red`, `Green`, `Blue`, and `Gray` are called the _constructors_ of the type `BasicColor`. The constructor `Gray` is said to be a parametrized constructor, while the other three are called indigenous constructors. The values of enumeration types can be inspected by exhaustive case analysis:
```scala
s match {
  case Red   => ...
  case Green => ...
  case Blue  => ...
  case Gray(intensity) => ...
}
```

**Definition 1**
> Variant data types are specified by a finite list of named constructors with a finite number (zero or more) parameters each. All parameters are required to have types (data types) defined beforehand. Values of variant datatypes are only allowed (1) to be created by manifestly using a constructor from the list, (2) to be inspected by case analysis.

Variant data types with indigenous constructors only are finite, i.e. variables of the respective types can attain only a finite number of values. This also applies to variant types with parametrized constructors as long as all parameters are of finite types.

The values of declarative data types have to be stored in the computer memory, and for that purpose they are mapped onto hardware-specific data structures. For instance, finite variant types can be stored as integers of sufficent bit size (`int8`, `int16`, `int32`), where each constructor is identified with a specific numerical value. In the example above, `BasicColor` could be stored as an `int8`, where `Red` could be assigned to -1 and `Green` to -2, `Blue` to -3, and the remaining 100 shades of gray to the numbers 0 to 100. This implementation is certainly non-unique. One could have chosen any other numerical codes or used a varable length encoding: four bits for the constructor variant, and additional 7 bits for the `intensity` field if the constructor happens to be `Gray`. The definition 1 requires that values are only created using constructors and inspected by case analysis, which renders it impossible to inspect which implementation is being used. This opaqueness does in particular allow the compiler to chose the implementation it pressumes to be optimal on the given machine in the given setting, or even to switch implementations depending on medium. For example, compact variable length encodings are often better for transporting data over the network, whereas fixed length encodings perform better in RAM.

§ Variant Types with bundled operations
---------------------------------------

There is a handy extension to variant types: one can allow constructors that reduce to other constructors. Reductible constructors are exempt to the rule that parameters must have types defind beforehand. In particular they are allowed to have parameters of the type being defined. Consider the following example:

**Example 2**
```
datatype Bool {
  True,
  False,
  Not(a : Bool),
  And(a : Bool, b : Bool),
  
  Not(True) => False,
  Not(False) => True,
  
  And(True, True) => True,
  And(True, False) => False,
  And(False, True) => False,
  And(False, False) => False,
}
```

Reducible constructors have to reduce to a non-reducible one for any combination of parameters.

Hardware-defined primitive data types such as `int32` and `float64` can be seen as finite variant types in disguise, because they can be modelled by finite sequences of bits:

**Example 3**
```
datatype Int8 {
  Int8(b1 : Bool, b2 : Bool,.., b8 : Bool),
  BitwiseAnd(a : Int8, b : Int8),
  BitwiseXor(a : Int8, b : Int8)
  Add(a : Int8, b : Int8),
  Mul(a : Int8, b : Int8),
  
  BitwiseAnd(Int8(a1,.., a8), Int8(b1,..,b8)) => Int8(And(a1, b1), And(a2, b2),.., And(a8, b8))
  
  ... // likewise define BitwiseXor and then addition via bitwise operations and multiplication via addition
}
```

§ Inductive types
-----------------

The realization that all low level primitive datatypes like `int32` are finite types, can mislead one into the conviction that all data types are. That is not true under assumption that computer memory is potentially infinite. The prime examples of infinite data types are the types of (unlimited) natural and integer numbers. It is certainly true, that there are integer numbers so large that they would not fit into the working memory of any given computer, but in such case it is potentially possible to build a computer with even larger memory.

To construct the types of natural and integer numbers declaratively, one needs an extension to the concept of variant types: the inductive types. In their simplest form, inductive types are cousins of variant types with recursive defintions allowed. That is, constructors of inductive types are allowed to have parameters of the type being defined:

**Example 4**
```c
datatype Nat {
  Zero,
  SuccessorOf(n : Nat)
}
```

The possible values of a variable of type `Nat` are thus `Zero`, `SuccessorOf(Zero)`, `SuccessorOf(SuccessorOf(Zero))`, etc. Cycles are, however, not allowed. In particular there can be no such `n : Nat` that `n = SuccessorOf(n)`.

For inductive datatypes, it makes sense to redefine indigenous constructors to include parametrized constructors as long as all their parameters are of the type being defined. Using this updated terminology, both constructors of `Nat` are indigenous.


Now recall what mathematical induction is: to prove a statement for all natural numbers, prove it for `Zero` and prove that whenever it holds for `n` it does also hold for for `SuccessorOf(n)`. For inductive types, exhaustive case analysis turns into a form of mathematical induction (hence, the name): in order to define a function on `Zero` and on `SuccessorOf(n)` under assumption that `f(n)` is already known.

**Example 5**
```
def isEven(n : Nat) : Boolean
  Zero => True
  SuccessorOf(n) => Not(isEven(n))
```

Functions defined in such a way are said to be structurally recursive. Acircularity of inductive types amounts to the property that structurally recursive functions always terminate, i.e. cannot fall into an endless loop.

Inductive types may have structurally recursive reducible constructors. Let us define some bundled operations for `Nat` to provide a solid example:

**Example 6**
```
datatype Nat {
  Zero,
  Succ(n : Nat),
  
  Add(n : Nat, m : Nat)
  Mul(n : Nat, m : Nat)
  
  Add(n, Zero) => n
  Add(n, Succ(m)) => Succ(Add(n, m))
  
  Mul(n, Zero) => Zero
  Mul(n, Succ(m)) => Add(n, Mul(n, m))
}
```

To define the type of integers, it is handy to allow partially reducible constructors:

**Example 7**
```
datatype Int {
  Pos(n : Nat),
  Neg(n : Nat),
  Zero,
  
  Pos(Zero) => Zero
  Neg(Zero) => Zero
}
```

When performing exhausitve case analysis, reducible cases do not appear. Let us define negation for integers to illustrate:


**Example 8**
```
datatype Int {
  Pos(n : Nat),
  Neg(n : Nat),
  Zero,
  Negate(n : Int),
  
  Pos(Zero) => Zero
  Neg(Zero) => Zero
  
  Negate(Zero) => Zero
  Negate(Pos(Succ(n))) => Neg(Succ(n))
  Negate(Neg(Succ(n))) => Pos(Succ(n))
}
```
Here, the cases `Pos(Zero)` and `Neg(Zero)` are not mentioned at all because `Negate` _has_ to reduce to `Negate(Zero)` in this case. 

Reducible and partially reducible constructors do not increase strength of basic inductive types, but they do increase strength of their further generalizations that will be considered later.


§ Quotient Inductive Types
--------------------------


Basic inductive types are not sufficient to handle the natural definition of rational types. Let us consider an extension to basic inductive types that makes them possible.

TODO

Tell that we're still to weak to encompass real numbers, but we have to wait before defining them.


§ Synthetic types
-----------------

Recall that variant types are not only finite if all their constructors are indigenous. They also remain finite with parametrized constructors as long as all of them do only have parameters of finite type. Similarily let us call an inductive type synthetic either if it has only indigenous constructors or if all paremeters of its parametrized constructors are synthetic as well.

Synthetic types are particularily well-behaved: each possible value of a synthetic type can be given as a finite tree of constructors. In particular, it means that synthetic types are effectively enumerable: one can write down an algorithm that prints out possible values of a given type and will eventually print out any of them.

Synthetic types that do not employ postulated identifications, equality of values is decidable, i.e. equality can be checked by an algorithm which is guaranteed to terminate. For synthetic types employing postulated identifications, the equality checking is only guaranteed to be verifiable, i.e. to terminate if the values are equal indeed. If the values are distinct, equality checking might run into an infinite loop. That is not a flaw of a particular equality checking algorithm, but an general problem known in mathematics as [word problem undecidability](https://en.wikipedia.org/wiki/Word_problem_(mathematics)).

TODO: few words about synthetic types and abstract syntax trees.


§ Function types
----------------

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

Along the same line of reasoning one can show that the type `A -> B` is a synthetic type in disguise if `A` is finite and `B` synthetic. The next section argues that this is the only case when function types are equivalent to synthetic types.

§ Non-Synthetic Types
---------------------

There are data types which cannot be expressed as synthetic types.  

The most prominent examples are given by functions on infinite data types (for instance infinite sequences of booleans `Nat -> Bool`) and exact real numbers `Real`. While this two types are perfectly valid data types, it is a matter of argument if infinite sequences and exact real numbers are data in the most narrow sense of the word because they cannot be stored on a finite digital carrier or sent over network in a finite amount of time.

Yet it is possible to define the types of _computable_ sequences and _computable_ real numbers. That is, ones sequences that can be produced by an algorithm and real numbers that can be algorithmically computed to any desired finite precision. These are unequivocally **data** types, because they can be easily sent over network or stored in form of the respective lambda expression (or any other desired form of Turing-complete computation). These two types still cannot be represented by closed synthetic types, because closed synthetic types are by design effectively enumerable and computable sequences/computable reals are not effectively enumerable due to [halting problem](https://en.wikipedia.org/wiki/Halting_problem).

The type of all lambda expressions can be certainly given by as a closed synthetic type, but it will neccesarily include some nonterminating expressions (the ones that does not correspond to any valid sequence and no valid real number respectively), and have the wrong notion of equality. As we already mentioned, equality on synthetic types is verifiable, while equality of reals and equality of sequences is not. Actually, it's the other way around. The equality on reals and discernable sequences is refutable: one can check equality of two numbers digit-by-digit (equality of sequences respectively item-by-item) and this process is guaranteed to terminate if there is a difference somewhere, but would last infinitely when values are indeed equal.

To have a mathematically sound type system, one has to accept the existence of open types of functions `A -> B` (where `A` is an infinite type) which are inhabited by constructively definable functions from `A` to `B`, but not limited to them. The only thing one can be sure, is that such a function can be applied to a value of type `A` and yields a value of type `B`. Function types inherently satisfy a potential open-world assumption.

§ The Power of Non-Synthetic Quotient Inductive-Inductive types: Defining reals
-------------------------------------------------------------------------------

Define reals, partial computations `℧(T)`, computable reals, computable functions, borel-measurable functions `𝔅(A, B)`.

Provide initial models for contably-infinitary algebraic theories (like the theory of compact Hausdorff spaces).


* * * 


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

§ Universes and other fibered inductive-inductive types
-------------------------------------------------------

Another kind of non-closedness arises when the functions defined simultaneously with their domain in inductive-inductive types are allowed to be kind-valued. Such extention is known as fibered inductive types (or large inductive-inductive-recursion). Since kinds are inherently open virtual types, resulting synthetic types are non-closed as well. This extension allows defining universes and abstract syntax trees for languages with extensible type systems. In particular, such types are capable of capturing abstract syntax of their host language, which is known as “language eating itself”. In the latter case, types remain positively semidecidable and effectively enumerable relative to the host language, which allows calling them syntactic models, at least in relativised sense.
