I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). We study a particular kind of type systems and their applications in programming languages and pure mathematics. This article is an attempt to explain our field to an interested programmer, who has some experience with a class-based programming language (like Java or C#)
and has seen some functional programming elements (perhaps in a language like Clojure, Scala, or F#).

Types are there to classify the range of variables and parameters in formal languages. Data types such as `int`, `List<int>`, etc. are the types you might be familiar with. Each formal language has its own type system, which guides its conceptual structure. However, most mainstream programming languages have ad hoc type systems that turn out to be inconsistent.

There are two types of formal languages: programming languages and the ones used for writing theorems and proofs. Our group works with both:
* We are working towards developing a sound type system for programming languages that exhibit complex computational behaviours including concurrency and non-determinism;
* We develop one of the leading interactive theorem provers called [Arend](https://arend-lang.github.io/) and its respective language.

The history of type theory is two-fold. Types were first introduced and studied by logicians and proof theorists, way before programmable computers came into existence. This part of the history began in a 1902 letter from Bertrand Russell to Gottlob Frege (both of whom were logicians), where types were introduced to solve an inconsistency in Frege's work, and culminated as Kurt Gödel (a proof theorist) developed a “programming language” of primitive recursive functionals to prove relative consistency of arithmetics by studying type theory of that language.  
From their side, programming language designers introduced types in the late 1950s for entirely unrelated reasons: variables and parameters had to have type declarations to tell the machine which register or how many memory cells to use for a given variable. Eventually statically typed programming languages with complex type systems emerged. To rectify incoherences of those ad hoc type systems, computer scientists rediscovered type theory in the 1970s. Since then, type theory has been developed by mathematicians and computer scientists hand in hand.

The practical goal of redesigning type systems of general-purpose programming languages is still far from being achieved. In fact, based on their experience with languages like C++, C# or Java, many programmers believe complex type systems to be a pointless pain in the neck. While type-theoretically sound languages (e.g. the ML family) are there for almost half a century, type systems of most mainstream languages are a type theorist's nightmare. However types are inevitable in programming languages, and carefully designing a type system in advance is the only way for typing not to be a nuissance.

There are several unrelated issues to be addressed:
* Mainstream languages tend to stick with bad typing practices where better ones are available;
* The gap between statically typed languages and dynamically typed languages has to be closed;
* There are computational behaviours for which good typing practices are yet to be determined.

The first issue seems to be due to inertia and a communication gap between computer scientists and engineers.

The static vs. dynamic typing gap can indeed be closed by means of gradual typing and type inference: mechanisms that allow omiting type annotations almost entirely in ?simple? cases. These mechanisms are indispensable for a language with a complex type system to have bearable learning curve and perform well at rapid prototyping. Gradual typing and type inference are readily available in some mainstream languages including C# and Scala. The interplay between gradual typing and other features of complex type systems is however highly nontrivial and not entirely understood yet. 

The issue our group primarily works on is the last one. Existing type-theoretically sound languages (ML family, Haskell etc.) are functional languages with limited or no support for desirable computational behaviours: concurrency, mutable state, and interaction with external actors. This is because all types present in these languages are data types, while aforementioned behaviours call for object types. This will be discussed below at length.


§ Types in Math and Types in Programming
----------------------------------------

To give the reader an idea of what a type system is, let's introduce the type system used by Gödel for his seminal relative consistency proof. It has a single primitive type `Nat` of natural numbers and types `Function<X, Y>` of explicitly computable functions accepting a value of type `X` and yielding a value of type `Y`. So has conists of types `Nat`, `Function<Nat, Nat>`, `Function<Function<Nat, Nat>, Nat>`, `Function<Function<Nat, Nat>, Function<Function<Nat, Nat>, Nat>>` etc.

In mathematics, type systems are used primarily in definitions of formal languages. A formal language is defined by a typed grammar that consists of rules like this:
```
 f : Function<X̲, Y̲>    x : X
—————————————————————————————
        f(x) : Y
```

A rule is a recipee for forming expressions. This rule introduces the notation `f(x)`. Constituents `f` and `x` annotated by their respective required types are written above the separating line, the resulting expression `f(x)` annotated by its type is written below the line. This rule reads “whenever you have an expression `f` of type `Function<X, Y>` for some types `X` and `Y`, and an expression `x` of type `X`, one can put them together to obtain an expression `f(x)` of type `Y`”. Here `f(x)` is not a pre-existing notation, it could bear any name instead, say `apply [f] to [x]` or `x |> f`.

Such grammar rules tell nothing about the intended meaning of expressions they define, they merely describe how expressions are composed. In particular, grammar rules can only be conditional on types of their constituent expressions. Consider the following non-example:
```
 p : Real   q : Real
–––––––––––––––––––——
    p / q : Real
```
Let us assume that `Real` denotes the type of real numbers. Then the expression `p / q` cannot be interpreted as division as there are cases for which properly typed `p` and `q` do not yield a real `p / q`. No condition like `q ≠ 0` can be imposed on constituents for it is not a condition on constituents' types. The only way for the resulting expression to be read as division, is to assign it a new type handling the division-by-zero case. For example, one could use a type (let's call it `PReal`) containing an additional value for the division-by-zero case.

Types as used in programming began in 1950s: type declarations were used by compilers to find out which registers or how many memory cells to use for respective variables, and which operations are permitted. The second usage seemingly coincides with mathematical usage of types, but in practice programming language designers used to treat typing rules as somewhat vague declarations of intent without objecting to declarations like `divide(x : Real, y : Real) : Real` (the non-example above).

Since, all variables in a low level programming language are bit strings of fixed length under the hood, minimalistic type systems have had a fixed finite number of built-in types directly corresponding to the hardware architecture of underlying systems, say `byte`, `int16`, `int32`, `real32`, `real64` and `pointer`. Yet as low level languages evolved, non-trivial type systems quickly emerged. There are two ways a type system can grow beyond a finite number of types: it either has to have type formers (= built-in types with compile-time parameters) or be extensible, so one can define entirely new types inside the program. Algol W (1966) already had both:
* types with compile-time parameters:
* * arrays of fixed length `n` and fixed element type `T` (`Array<T, n>` in C++-esque notation) in Algol 60
* * pointers annotated by the type of variable they point to (`Pointer<T>` in C++-esque notation) in Algol W
* user-defined types for typed records (also known as structures).

Most modern statically typed mainstream languages straightforwardly extend type system of Algol W allowing for domain-specific data types (say, `Date` or `Color`) and custom data structures like `List<SomeType>`, `BinaryTree<SomeType>`, `Collection<SomeType>` and `Map<KeyType, ValueType>`.


§ What's wrong with C-style type systems?
-----------------------------------------

Already a finite type system can bear problems. When a compiler encounters a case where a value of the type `X` is used in a context where values of the type `Y` is required, it can either terminate with an error message or apply an implicit conversion from `X` to `Y`. For example, most (if not all) compilers would silently agree to use an `int16` value in a context where an `int32` is required: they're simply pad the binary number representation to match the length. Some languages also implicitly convert `int16` to `float32` and `int32`s to `float64` since no information loss takes place. Here the first two problems arise:

1) There can be more than one way to convert `A` to `B`, say `int16 -> int32 -> float64` and `int16 -> float32 -> float64`. They have to be equivalent. If your language has extensible type system and supports custom implicit conversions (which is the case in Scala), you have to enforce this property somehow (which cannot be done in Scala).

2) If the language supports overloading (i.e. the same expression may be interpreted differently depending on types of operands), result of the expression should stay the same regardless of implicit conversions. For example in C `a / b` means integer division if `a` and `b` are integers, or real division if `a` and `b` are floating point approximations of real numbers. In this example `1 / 2 = 0` if `1` and `2` are interpreted as integer numbers, but `0.5` if they are interpreted as floats: a clear case of incoherent behavior, which can be easily fixed by using a diffrent operator for integer division (like `//` in Python). There are however more subtle cases: assume, you use `+` both for unguarded addition of `int16`s and `int32`s. Now if you add two large 16 bit integers as `int16`s you'll obtain a negative number, whereas adding them as `int32`s would yield a postive number. In case you use `+` for guarded addition of both `int16`s and `int32`s you'll get an `#OVERFLOW` in the first case and a proper number in the second one. You cannot have overloaded arithmetic operators in a low level language with implicit conversions without being incoherent! (In a high-level language you can have a proper addition operator `+`, for instance in Python `+` performs addition of fixed length integers by default, yet performs automatic fallback to arbitrary precision integer representation on overflow and underflow.  
It's extremly hard to have both overloading and implicit conversions together without being incoherent!

When type formers come into play, it is quite natural to automatically extend implicit conversions along covariant parameters: implicit convertibility from `X` to `Y` should imply implicit convertibility from (immutable) `List<X>` to `List<Y>` and from (pure) `Function<T, X>` to `Function<T, Y>`. But in can only apply to some parameters of type formers, namely so called covariant parameters. Some programming languages (prime example being Scala) also derive implicit convertibility along contravariant parameters, not only from (immutable) `Map<T, X>` to `Map<T, Y>`, but also from `Map<Y, T>` to `Map<X, T>` (attention, reversed order!).

Such conversions should be available on demand (i.e. as explicit coersions), but not performed implicitly for they can lead to unintentional information loss.

* * *

§ Strictly Typed Languages
--------------------------

There are several programming languages (Haskell being the most prominent example) that reconcile the mathematical notion of types with the programmers' notion: the strictly typed languages. NB!: Strictly typed does not mean statically typed: a strictly typed language can support type inference and gradual typing (i.e. “duck typing” where possible until explicit type anotations are provided): public APIs, mature libraries and safety-critical software probably should be explicitely typed in the most precise manner, but early prototypes and unsettled components should be as concise and agile as possible.

Strictly typed does not mean purely functional either: most strictly typed languages are, but it is for the reason that type systems for mixed paradigm programming are still very much work in progress, while type systems for purely functional ones are rather humble extensions of systems developed by mathematicians (Alonzo Church, Kurt Gödel, et al.) over half a century ago, for the most part way before any computers even existed.

The field of strictly typed languages is still in its infancy. We have not yet completely figured out how type complex computational behaviours at all, let alone doing it conveniently, comprehensibly for non-experts, and with a transparent mental model capable of estimating performance.


§ What's wrong with Java-like type systems?
-------------------------------------------

Here we'll discuss widespread flaws in type systems of statically typed class-based object oriented programming languages as exemplified by Java.

Java has a small set of primitive data types (boolean, int and float) which cannot be extended, and an extensible system of reference data types. In Java and all JVM-languages, the type `int` (integer) is misnomer as a tribute to the C programming language. Java `int`-values are limited to numbers between -2'147'483'648 and 2'147'483'647 without any overflow or underflow protection by default. There are some higher level languages like Python (which has quite a different type system), whenever an integer over- or underflows, i.e. fails to fit into the register or memory cell it is intended to be stored, automatic fallback to arbitrary precision integer representation occures, a technique that can be implemented without any performance losses (in case no over-/underflow ever occures) on many CPU architectures.

Primitive data types classify values (thus are _data_ types indeed) that can be passed as arguments, stored as local variables or in fields (“typed memory cells”) of (mutable) objects stored in the heap. Values of primitive data types are always passed call-by-value.

Reference data types classify objects (in general, mutable) objects stored in heap or being external resources such as files, IO-streams, remote services or devices like printers or cameras. Arguments of reference types are always passed call-by-name (aka call-by-reference). In Java, there are two kinds of reference types: array types (references to fixed-length mutable arrays of values or references of the same type) and object types also called classes. One can define custom classes, but there are some predefined ("inbuilt") ones like “Object”, “Thread”, “File”, etc. In many Java-like systems, arrays are objects as well, i.e. array types are just inbuilt classes, leading to a way more uniform type system. Unfortunatelly, in Java any reference is allowed to take the special value `null` without any prior warning. (There are some JVM languages remedying both issues.)

Java does not provide any way to define additional data types: types which would classify stuff that is passed call-by-value and contains only self-contained data, in particular no references to (possibly mutable) objects or external resources. But datatypes can be emulated using a special trick: when defining a class (type of possibly mutable object), one can declare one or more fields immutable. Let's call a class recursively immutable if all its fields are declared immutable and have either primitive or recursively immutable reference types. Under additional assumptions that nullability of refrerences can be limited, one can even express some nontrivial data structures, e.g.
```Java
@RecursivelyImmutable
final class LinkedList<@RecursivelyImmutable T> {
  final @NonNullable T head;
  final @Nullable LinkedList<T> tail;
}
// Note: We cannot rule out circular lists in Java.
```

Recursively immutable classes represent the most mature part of Javaesque type systems, but let me show that even they are flawed.

* * *

Let me first name a few problems and then discuss them in detail:
- Lack of structurally typed (aka duck-typed) tuple and record types;
- Implicit contravariant casts are fundamentally flawed;
- Subtyping defined by inheritance fundamentally flawed;
- Both subtyping and inheritance clash with mutability;
– No way to define pure functions (those with no side effects);
– No way to define pure data structures (those with no circularities);
– Generics inconsistent;
– Datatype-generic programming not available (impossible to define, say, a `serialize()`-method for generically for all possible classes containing serializable fields only).



§ Sorting the wheat from the chaff: Values, objects and abstract parameters
---------------------------------------------------------------------------

There are three classes of types.
– Data types like `Int`, `List[Int]`, `Real` or `Int -> Int` (a "mathematical" function from integers to integers, i.e. total deterministic pure function). These are types of values, pure information that can be copied or discarded arbitrarily without any caveats, it can be sent in a message over the network or saved onto a disk. Whenever one writes `f(x̲ : Int)` it is meant that `f` takes an integer value as an argument. As opposed to languages like Java and C#, in strict languages `List[Int]` is always to be understood as a(n inherently immutable) value, not as a mutable container of integers. In case `f(x̲ : List[Int])` the variable `x` refers to a value upon which `f` was called, it cannot be modified by a third party in concurrent setting or used to export modifications back to the call site, it's just a value. In a strict language, the notation `x := x + 1` is either completely prohibited in order to avoid confusion (one has to use a fresh name each time, `y := x + 1`) or just shadows an existing local constant `x` by a new one, leaving old one accesible outside of the scope where the new definition took place. It is, `x := x + 1` if it is allowed, is just a syntactic sugar, defined to work exactly the same way as if the left `x` were replaced by a fresh variable together with all usages of `x` in the same scope below this definition. The compiler might internally implement this using mutability if the overshadowed constant is never used again, but on the semantic level there is no mutability here. Immutability of values (or broadly speaking, referential transparency, the fact that expressions might be freely replaced by their values, after all variables were disambiguated to have unique names) allow flexible execution order: any mixture of lazy, eager and preemptive, out-of-order and parallel is guaranteed to produce same results and no side effects.

– Object types classify objects which cannot be cloned or discarded silently in general. The examples being suspended computations (aka continuations), actors (possibly with encapsulated mutable state), and external resources (say a file handle or a quantum register). These are written `f(1x̲ : File)`, the prefix 1 reflecting that they have to be used precisely once in every execution branch of the function body. For our example of a file, it can be closed, modified (in this case the modification routine consumes `x` and yields a new file handle referring to the modified file), transfered to another owner or returned back in some form. Here again some languages might allow to use the same name for the new file handle if it introduces no ambiguity. Linearity of usage of such variables introduces some sequentiality into computations (impose partial execution order), but exactly the right amount: computations with values remain flexible and only have to synchronize on interactions with objects; interactions with objects encapsulate side effects including IO, communications between processes/actors. A function defined in a context with one or more objects is not a pure function anymore, it's should be better called a procedure or a process.

— Besides (value) arguments and object arguments, functions might have abstract parameters, these are never used in the body of the function but are used for generic programming, i.e. to write down the correct types of some arguments and/or the return value, and possibly to impose some conditions onto the arguments. These are prefixed by a zero. To give an example, lets write down a signature of a function applying a function `f` to each element of a list `l`.
```
map(0X̲ 0Y̲ : *, 0n̲ : Nat, l̲ : List[X, length = n], f̲ : X -> T) : List[Y]
```

Here we additionally guarantee that this operation preserves the lenth of the list. In the above example we have three abstract parameters: the datatypes `X` and `Y` of elements of the inbound and outbound list respectively and a natural number `n`: the length of the both lists. The list `l` and the function `f` are ordinary (value) arguments. As you can see, abstract parameters can be of an ordinary datatype (like `Int`), but additionally they might be of a purely abstract type like the type of all datatypes `*`, type of all datatypes parametrized by a datatype `* -> *` (like `List[_]`), a datatype parametrized by a number `Nat -> *`, a datatype parametrized by a sequence of types `(Nat -> *) -> *`. For the mathematicians among us, also a function might also require a datatype together with a group structure on that datatype `(0T̲ : *, g̲ : Group[T])`, a datatype together with another datatype doubly parametrized by it and a structure of category on them: `(0O̲b : *, 0M̲ap : Ob -> Ob -> *, cat : Category[Ob, Map])`, analogously we can also write down the purely abstract type of functors between categories and natural transformations between functors, and so on. While objects and object types have apparently no applications in a language for theorems and their proofs, abstract parameters and purely abstract types play a pivotal role: they allow for generic theorems and proofs, applying to all groups, all categories etc without size restrictions.

Abstract parameters by definition enjoy a property called erasability: they are irrelevant for runtime execution, they are there only for compile-time type checking and elaboration.

Type systems supporting abstract parameters (at least of the type `*`) are called polymorphic, the ability to handle purely abstract types like `* -> *` and `* -> * -> *` is known as higher-kinded polymorphism, handling `(Nat -> *) -> *` is known as complex kinded polymorphism. If types of further arguments and the return type are allowed to depend not only on abstract parameters, but also on ordinary values, the type system is told to feature dependent types (a part of the name of our research group). 


§ Generating all data types: Synthetic types and Function types
---------------------------------------------------------------

It turns out, that all data types that have found applications in programming and concrete mathematics to this day, including even unbounded arbitrary-precession real numbers, can be defined combining synthetic datatypes (the ones susceptible to pattern matching, see below) and function types `X -> Y`, the types of “mathematical” (i.e. pure, total, deterministic) functions turning values of the type `X` into values of the type `Y`, or their dependent form where the result type can depend on the input value (denoted by `(x̲ : X) -> Y[x]` or alternatively `∀(x̲ : X), Y[y]`).

Function types `X -> Y` follow the open-world assumption: it is assumed that the type of functions includes all functions definable by terminating exhaustive pattern matching, but not limited to them (because we know that there are more computable functions than one can show to be terminating in any given fixed theory). The only thing we assume is that any `f : X -> Y` can be applied to a value `x : X` yielding a value `f(x) : Y`. In presence of abstract interval type (see next section) we can also show that any two functions that agree on all inputs are equal and therefore indistinguishable: the typesystem itself prohibits syntactic introspection into function types, if they are defined as described. Please note that Haskell and many similar languages lack proper function types: by `X -> Y` they denote the type of computable partial functions which are not guaranteed to terminate. Partial function types are not sufficient for many applications we'll be talking below.

Synthetic datatypes generalise finite datatypes, the so called enumerations:
```
@Data Cardsuit:
  Clubs
  Diamonds
  Hearts
  Spades
```

Here, `Clubs`, `Diamonds`, `Hearts` and `Spades` are called constructors of the cardsuit types. Synthetic datatypes allow constructors to have parameters, say
```
@Data CalcConstant:
  Pi
  EulerNumber
  Literal(n̲ : Int)
```

Futhermore, synthetic datatypes allow parameters to be of the type being currently defined as long as recursion stays well-founded:
```
@Data Nat:
  Zero
  Succ(n̲ : Nat)
```

If all constructor parameters of a synthetic type are of a syntethetic type themselves all the way down, the type is called closed synthetic type. In this case all possible values of the synthetic type can be exhaustively enumerated, in particular synthetic types follow the closed-world assumption, all values can be listed and there are no values besides that, equality of two values is decidable by a trivial top-down recursive algorithm. Functions on synthetic types (not necessary closed ones) can be defined by means of structural induction/recursion on the constructors of the type (i.e. essentially just pattern matching). Under mild assumptions easily checkable by the compiler they can be shown to be computable and terminating by definition.

Mixed synthetic types (non-closed ones) do not follow the closed world assumption anymore since some of the constructors may have functions (following open-world assumption) as parameters. Constructors with parameters of abstract interval type enable synthetic types with custom equality (see below). The type of arbitrary precision real numbers is a very complex mixed synthetic type with custom equality. For the mathematicians among us, closed synthetic types with custom equality (also known as higher inductive-inductive types) correspond to initial algebras for finitary generalized algebraic theories without equations on sorts. In case of mixed types (like reals) one goes beyond finitarity.

§ Extentsions: induction-recursion and coinductivity
----------------------------------------------------

§ Semi-simplicial Types
-----------------------

```
@Structure SSType:
   headT : U
   tailT : dec(headT) -> SSType
```

```
@Structure VDF[T : SSType]:
  head : dec(T.headT)
  tail : VDF[T.tailT(head)])
  
@def apply(f : VDF[T], n : Nat):
   f(.tail)^n.head
```

§ Abstract interval type
------------------------


§ Purely abstract types and universes: strong reflection principle
------------------------------------------------------------------
