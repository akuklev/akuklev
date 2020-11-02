I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). We study particular kind of type systems and their applications in programming languages and pure mathematics. This article is an attempt to explain our field to an interested programmer. I assume our intended reader to have some experience with a low-level programming language like C, a “strongly” typed object-oriented language supporting generics like Java or C#, and having seen some functional programming elements, perhaps in a language like Clojure, Scala or F#. Proficiency in strictly typed purely functional programming languages like Haskell or fluency in advanced math is _not_ assumed.

Types are there to classify “range” of variables and parameters in two classes of formal languages: programming languages and mathematical languages used for writing down theorems and proofs. Our groups is works in both directions:
– We develop one of the leading interactive theorem provers called [Arend](https://arend-lang.github.io/) and its respective language.
— We are working on type system embracing complex computational behaviours including concurrency and nondeterminism.

History of type theory is therefore twofold, types were first introduced and studied by logicians and proof theorists, most notably Bertrand Russell between 1902 and 1908 for the family of formal languages for his Principia Mathematica project of codifying basic mathematics, Alonzo Church around 1935 for his proof that first order logic is in general undecidable and Kurt Gödel between 1944 and 1958 for his proof of relative consistency of arithmetics.

Types were rediscovered by programming language designers in late 1950s, but the type systems used for programming languages were rudimentary and incomplete. Type declarations were used only by compilers to find out what registers/how many memory cells to use for which variables, and which operations are allowed on which variables. You don't need a complex type system for that: under the hood all variables were bit strings of fixed length. Early programming languages even had a fixed number of data types directly corresponding to the hardware architecture of underlying systems, say `byte`, `int16`, `int32`, `real32`, `real64` and `pointer`. In programming languages like C++, Java, C# and alike, you have complex type systems allowing for both domain-specific data types (say, `Date` or `Color`) and custom containers like `List<SomeType>`, `BitTree<SomeType>`, `Collection<SomeType>` and `Map<KeyType, ValueType>`, but these type systems are still incomplete and don't have self-sufficient intrinsic definitions independent of hardware implementations of their respective programming languages. Incompleteness refers to the fact that sometimes you need to forcibly “cast” values between types to write perfectly sensible programs. Such types only annotate variables facilitating some syntactic sugar and a few very weak correctness guarantees checkable in compile time, while being a leaky abstraction "pegged" upon the true operational semantics of their respective programming language. Type systems we study are _not_ of that kind.


§ Strictly typed languages
--------------------------

A handful mostly academic programming languages (Haskell being the most prominent example) reconcile the mathematical notion of types with the programmers' notion. Such languages are calles strictly typed.

Strictly typed does not mean statically typed: a strictly typed language can support type inference and/or gradual typing, both of which are highly desirable for rapid prototyping: public APIs, mature libraries and safety-critical software probably should be explicitely typed in the most precise manner, while early prototypes and unsettled components should be as concise and agile as possible.

Strictly typed does not mean purely functions: most strictly typed languages are, but it is for the reason that type systems for mixed paradigm programming are still very much work in progress, while type systems for purely functional ones are rather humble extensions of systems developed by mathematicians (Alonzo Church, Kurt Gödel, et al.) over half a century ago, for the most part way before any computers even existed.

All strictly typed programming languages known to the author at the moment are rather poor at rapid prototyping, have steep learning curves and extremely involved cost models rendering any performance optimization whatsoever virtually inaccessible to non-experts, which is the reason why these are non-mainstream, mostly academic programming languages. We are sure that all of these setbacks can be dealt with, yet it will surely take a decade or two.


§ Values, objects and abstract parameters
-----------------------------------------

There are three classes of types.
– Data types like `Int`, `List[Int]`, `Real` or `Int -> Int` (a "mathematical" function from integers to integers, i.e. total deterministic pure function). These are types of values, pure information that can be copied or discarded arbitrarily without any caveats, it can be sent in a message over the network or saved onto a disk. Whenever one writes `f(x̲ : Int)` it is meant that `f` takes an integer value as an argument. As opposed to languages like Java and C#, in strict languages `List[Int]` is always to be understood as a(n inherently immutable) value, not as a mutable container of integers. In case `f(x̲ : List[Int])` the variable `x` refers to a value upon which `f` was called, it cannot be modified by a third party in concurrent setting or used to export modifications back to the call site, it's just a value. In a strict language, the notation `x := x + 1` is either completely prohibited in order to avoid confusion (one has to use a fresh name each time, `y := x + 1`) or just shadows an existing local constant `x` by a new one, leaving old one accesible outside of the scope where the new definition took place. It is, `x := x + 1` if it is allowed, is just a syntactic sugar, defined to work exactly the same way as if the left `x` were replaced by a fresh variable together with all usages of `x` in the same scope below this definition. The compiler might internally implement this using mutability if the overshadowed constant is never used again, but on the semantic level there is no mutability here. Immutability of values (or broadly speaking, referential transparency, the fact that expressions might be freely replaced by their values, after all variables were disambiguated to have unique names) allow flexible execution order: any mixture of lazy, eager and preemptive, out-of-order and parallel is guaranteed to produce same results and no side effects.

– Object types classify objects which cannot be cloned or discarded silently in general. The examples being suspended computations (aka continuations), actors (possibly with encapsulated mutable state), and external resources (say a file handle or a quantum register). These are written `f(1x̲ : File)`, the prefix 1 reflecting that they have to be used precisely once in every execution branch of the function body. For our example of a file, it can be closed, modified (in this case the modification routine consumes `x` and yields a new file handle referring to the modified file), transfered to another owner or returned back in some form. Here again some languages might allow to use the same name for the new file handle if it introduces no ambiguity. Linearity of usage of such variables introduces some sequentiality into computations (impose partial execution order), but exactly the right amount: computations with values remain flexible and only have to synchronize on interactions with objects; interactions with objects encapsulate side effects including IO, communications between processes/actors. A function defined in a context with one or more objects is not a pure function anymore, it's should be better called a procedure or a process.

— Besides (value) arguments and object arguments, functions might have abstract parameters, these are never used in the body of the function but are used for generic programming, i.e. to write down the correct types of some arguments and/or the return value, and possibly to impose some conditions onto the arguments. These are prefixed by a zero. To give an example, lets write down a signature of a function applying a function `f` to each element of a list `l`.
```
map(0X̲ 0Y̲ : *, 0n̲ : Nat, l̲ : List[X, length = n], f̲ : X -> T) : List[Y]
```

Here we additionally guarantee that this operation preserves the lenth of the list. In the above example we have three abstract parameters: the datatypes `X` and `Y` of elements of the inbound and outbound list respectively and a natural number `n`: the length of the both lists. The list `l` and the function `f` are ordinary (value) arguments. As you can see, abstract parameters can be of an ordinary datatype (like `Int`), but additionally they might be of a purely abstract type like the type of all datatypes `*`, type of all datatypes parametrized by a datatype `* -> *` (like `List[_]`), a datatype parametrized by a number `Nat -> *`, a datatype parametrized by a sequence of types `(Nat -> *) -> *`. For the mathematicians among us, also a function might also require a datatype together with a group structure on that datatype `(0T̲ : *, g̲ : Group[T])`, a datatype together with another datatype doubly parametrized by it and a structure of category on them: `(0O̲b : *, 0M̲ap : Ob -> Ob -> *, cat : Category[Ob, Map])`, analogously we can also write down the purely abstract type of functors between categories and natural transformations between functors, and so on. While objects and object types have apparently no applications in a language for theorems and their proofs, abstract parameters and purely abstract types play a pivotal role: they allow for generic theorems and proofs, applying to all groups, all categories etc without size restrictions.

Abstract parameters by definition enjoy a property called erasability: they are irrelevant for runtime execution, they are there only for compile-time type checking and elaboration.

Type systems supporting abstract parameters (at least of the type `*`) are called polymorphic, the ability to handle purely abstract types like `* -> *` and `* -> * -> *` is known as higher kinded polymorphism, handling `(Nat -> *) -> *` is known as complex kinded polymorphism. If types of further arguments and the return type are allowed to depend not only on abstract parameters, but also on ordinary values, the type system is told to feature dependent types (a part of the name of our research group). 


§ Synthetic types and Function types
------------------------------------

While data types in C-like and Java-like languages essentially classify bit strings, data types in 

```
@Structure SSType:
   get : (T : U, dec(T) -> SSType)
```

```
@Structure VDF[T : SSType]:
  get : (head : dec(fst T.get), tail : VDF[(snd T.get)(head)])

```
