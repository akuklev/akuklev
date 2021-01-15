I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). We study a particular kind of type systems and their applications in programming languages and pure mathematics. This article is an attempt to explain our field to an interested programmer, who has some experience with a class-based programming language (like Java or C#)
and who has seen some functional programming elements (perhaps in a language like Clojure, Scala, or F#).

§ Types: a Brief Introduction
-----------------------------

Data types such as `int`, `List<int>`, etc. are the types you might be familiar with. Besides programming languages, types are also present in formalized languages for mathematical theorems and their machine-checkable proofs. Semantically, type is a space of possible values (or more generally, possible objects) a variable can refer to. Syntactically, type is a label associated with a variable on its declaration that determines available operations and specifies the behaviour of these operations. Each formalized language has its own type system, which guides its conceptual structure. Languages, where semantic and syntactic aspects of types are in harmony, are called strictly typed languages. At the moment, mainstream programming languages are not strictly typed, but have ad hoc type systems. In most cases these type systems are also incoherent in one way or another.

The history of type theory is two-fold. Types were first introduced way before programmable computers came into existence, namely, in a 1902 letter from Bertrand Russell to Gottlob Frege (both of whom were logicians). There, types were introduced to solve an inconsistency in Frege's work developing a formalized language for mathematical foundations. Logicians have studied and used type theory ever since. In the 40s and 50s Kurt Gödel (a proof theorist) developed a “programming” language of primitive recursive functionals to prove relative consistency of arithmetics by studying type theory of that language. While Gödel never intended to run this language on a real machine, this was the first example of a strictly typed programming language. In fact, nearly all modern strictly typed functional languages are its direct descendants.

Programming language designers introduced types in the late 1950s for entirely unrelated reasons: variables and parameters had to have type declarations to tell the machine which register or how many memory cells to use for a given variable. Eventually, programming languages with complex type systems emerged. To rectify incoherences of those ad hoc type systems, computer scientists rediscovered mathematicians’ work on type theory in the 1970s. Since then, type theory has been developed by mathematicians and computer scientists hand in hand.

The practical goal of redesigning type systems of general-purpose programming languages is still far from being achieved. In fact, based on their experience with languages like C++, C# or Java, many programmers believe complex type systems to be a pointless pain in the neck. While type-theoretically sound languages (e.g. the ML family) are there for almost half a century, type systems of most mainstream languages are a type theorist's nightmare. However, types are inevitable in programming languages, and carefully designing a type system in advance is the only way for typing not to be a nuisance.

There are several unrelated issues to be addressed:
1) Mainstream languages tend to stick with bad typing practices where better ones are available;
2) The gap between statically typed languages and dynamically typed languages has to be closed;
3) There are computational behaviours and programming techniques for which good typing practices are yet to be determined.

The first issue seems to be due to a communication gap between computer scientists and engineers, and huge systemic inertia: one cannot simply start from scratch, there is a necessity to retain backwards compatibility, there are lots of legacy systems, and there are many millions of software developers, who cannot learn unfamiliar programming languages and adopt new practices at once. Every serious shift is a tremendous effort to the industry of that size. We hope that availability of nice general-purpose languages where types are not a nuisance but an aid would help solving this issue, especially if those languages would be attractive for educators.

The static vs. dynamic typing gap can indeed be closed by means of gradual typing and type inference: mechanisms that allow omitting type annotations almost entirely in tractable cases. One still wants to use type annotations for public APIs, settled libraries and critical sections where you need strong correctness guarantees, as well as for type-driven development. Yet, omitting them is indispensable for a language with a complex type system to have a bearable learning curve and perform well at rapid prototyping. Gradual typing and type inference are readily available in some mainstream languages including C# and Scala. The interplay between gradual typing and other features of complex type systems is, however, highly non-trivial and not entirely understood yet.

The issue our group works on is the last one, that is, developing good typing practices for complex computational behaviours and programming techniques.  
Most existing type-theoretically sound languages are purely functional languages (such as Standard ML, Haskell etc). That is not because good type systems are unavailable for other languages, but rather because purely functional languages require much simpler type systems. In purely functional languages, variables always refer to values, i.e. immutable pieces of data. Сonsequently, their type systems have to deal only with _data_ types which are well understood. Here we use the term data types in this strict sense. On the other hand, types of varables referring to mutable objects and non-clonable resources (such as files, IO-streams, remote services or devices) are called _entity_ or _object_ types. Proper handling of object types is much more complex than that of data types. However, innate treatment of concurrency, mutable state and communication neccessarily requires using entity types besides data types.

There is also an advanced programming technique of generic programming, coarse form of which is available as “templates” in C++ and “generics” in Java. Quoting Bjarne Stroustrup: “Following \[Alexander\] Stepanov, we can define generic programming without mentioning language features: Lift algorithms and data structures from concrete examples to their most general and abstract form.” In theory, generic programming would allow to get around boilerplate code entirely. However this promise is yet to be fulfilled in a real programming language. Generic programming is available in Scala programming language in quite advanced yet rather unsatisfactory form, mainly due to lack of innate support on the level of typing. The proper handling of generic programming calls for yet another class of types: the _purely abstract_ types, which are types of “compile-time” parameters.

Both _entity_ and _purely abstract_ are crucial for developing good typing discipline for desirable computational behaviours and programming techniques and will be discussed at length below.

Before proceeding to these questions, let us breefly consider how types are used in mathematics and programming, and discuss several most distrubing type-theoretic incoherences in mainstream programming languages.

§ Types in Math
---------------

To give the reader an idea of what a type system is, let's introduce the type system used by Gödel for his seminal relative consistency proof. It has a single primitive type `Nat` of natural numbers and types `Function<X, Y>` of explicitly computable functions accepting a value of type `X` and yielding a value of type `Y`. Here are some examples of types in this type system: `Nat`, `Function<Nat, Nat>`, `Function<Function<Nat, Nat>, Nat>`, `Function<Function<Nat, Nat>, Function<Function<Nat, Nat>, Nat>>` and so on.

In mathematics, type systems are used primarily in definitions of formalized languages. A formalized language is defined by a typed grammar that consists of rules like this:
```
 f : Function<X̲, Y̲>    x : X
—————————————————————————————
        f(x) : Y
```

A rule is a recipe for forming expressions. This rule introduces the notation `f(x)`. Constituents `f` and `x` annotated by their respective required types are written above the separating line, and the resulting expression `f(x)` annotated by its type is written below the line. This rule reads “whenever you have an expression `f` of type `Function<X, Y>` for some types `X` and `Y`, and an expression `x` of type `X`, one can put them together to obtain an expression `f(x)` of type `Y`”. Here `f(x)` is not a pre-existing notation, it could bear any name instead, say `apply [f] to [x]` or `x |> f`.

Such grammar rules tell nothing about the intended meaning of expressions they define, they merely describe how expressions are composed. In particular, grammar rules can only be conditional on types of their constituent expressions. Consider the following non-example:
```
 p : Real   q : Real
–––––––––––––––––––——
    p / q : Real
```
Let us assume that `Real` denotes the type of real numbers. Then the expression `p / q` cannot be interpreted as division as there are cases for which properly typed `p` and `q` do not yield a real `p / q`. No condition like `q ≠ 0` can be imposed on constituents for it is not a condition on constituents' type. The only way for the resulting expression to be read as division, is to assign it a new type that handles the division-by-zero case. For example, one could use a type (let's call it `PReal`) containing an additional value for the division-by-zero case.

§ Types in Programming
----------------------

Types appeared in programming in 1950s: type declarations were used by compilers to find out which registers or how many memory cells to use for respective variables and which operations are permitted. The second usage seemingly coincides with the mathematical usage of types, but in practice programming language designers used to treat typing rules as somewhat vague declarations of intent. For example, in many languages type declaration `divide(x : Real, y : Real) : Real` (the non-example above) is perfectly valid.

To illustrate how types were used in early programming languages, let us consider the following. Under the hood, all variables in low-level programming languages are bit strings of fixed length. Therefore, minimalistic type systems have had a finite collection of built-in types directly corresponding to the hardware architecture of underlying systems, say `byte`, `int16`, `int32`, `real32`, `real64` and `pointer`. Yet, as low-level languages evolved, non-trivial type systems quickly emerged.

There are two ways a type system can grow beyond a finite number of types: it either has to have type formers or to be extensible. Type formers are built-in types with compile-time parameters like `Function<X, Y>` we saw above. Type system extensibility means one can define entirely new types inside a program.

There is a common misconception, that complex type systems are a recent invention. However, a rather early programming language, Algol W, released in 1966, already had both features mentioned above:
* it had type formers, in particular:
* * arrays of fixed length `n` and fixed element type `T` (`Array<T, n>` in C++-esque notation);
* * pointers annotated by the type of variable they point to (`Pointer<T>` in C++-esque notation);
* its type system was extensible: it supported user-defined types for typed records (also known as structures).

Type systems of most modern mainstream languages have not gone far beyond Algol W. The only essential development is more advanced extensibility. Modern languages typically allow user-defined domain-specific data types (say, `Date` or `Color`) and type formers. The latter are typically used to provide custom data structures like `List<ItemType>`, `BinaryTree<ItemType>`, `Collection<ItemType>` and `Map<KeyType, ValueType>`.


§ Digression: What's wrong with C-style type systems?
-------------------------------------------------------

Type system does not have to be complex to be incoherent. Even a finite type system can have nontrivial issues when implicit type conversions come into play.

Let us illustrate the notion of implicit type conversion with a simple example: the user applies a function `f(n : int32)` to a value `v` of type `int16`. The compiler has two options in this case:
* Terminate with an error message “type mismatch on line ...”;
* Implicitly convert `v` of type `int16` into an `int32` (by simply padding its binary representation).

In the first case, the user has to explicitly convert values to match the required type every time. Obviously one would like to avoid being that verbose. Implicit conversions are, thus, just a matter of convenience, but they are present in most programming languages. In those languages, the compiler (or the interpreter) has an additional elaboration phase, where it transforms implicit conversions into explicit ones.

One has to be very careful about implicit conversions. They can easily violate the principle of least astonishment that states that every system should behave in a way non-expert users reasonably expect it to behave. The typical example of such a violation is notorious `1 / 2` ≠ `1 / 2.0` in C and its descendents.

There are general issues tied to implicit conversions across the languages that use them. Besides that, in class based languages (C++, Java, etc.) there are additional issues that arise from interplay of implicit conversions and inheritance.

The general issues implicit conversions often introduce are:
* Accidental information loss;
* Elaboration ambiguities;
* Interference with operator overloading;
which can be thought of as different conceptualizations of the same underlying issue.

§§ Accidental information loss
------------------------------

Let us begin with the most obvious issue of accidental information loss. Consider a language with an implicit conversion from `int32` to `float32`. While `int32` can store numbers between -2³² and 2³²-1, `float32` is capable representing numbers between approx. -2³⁸ and 2³⁸, thus such a conversion never leads to an overflow or an underflow. Yet, it leads to a precision loss: many 32 bit integers, say 20000000 and 20000001 would be represented by the same `float32`. Such conversion does not render the program obviously wrong: many programs would still calculate sensible results. Yet, it leads to information loss that easily goes unnoticed while leading to wrong outcomes.

It gets even trickier when types with parameters like `List<T>` and `Map<K, T>` come into play. A parameter `T` of a data type `SomeType<T>` is called covariant if there is a canonical way to map functions `X -> Y` onto functions `SomeType<X> -> SomeType<Y>` that preserves function composition. For example, the parameter `T` of `List<T>` is covariant: the canonical way to turn a function `f : X -> Y` into a function on lists `g : List<X> -> List<Y>` is to apply `f` to each element of the original list of type `List<X>`.

Let us consider a more complex example. The type `Map<K, T>` (of key-value dictionaries) can be seen as a generalization of type `List<T>`. `Map<K, T>`, just like `List<T>`, contains a finite collection of items of fixed type `T`. The difference is that the items of `Map<K, T>` are indexed not by consecutive numbers, but by keys of type `K`. Similarily to the case of lists, functions `X -> Y` can be applied to `Map<K, X>` itemwise yielding a `Map<K, Y>`. Thus, the second parameter (`T`) of `Map<K, T>` is covariant. However, the first parameter, `K`, is not, which will be importaint for the example below.

It is quite natural to extend implicit conversions along covariant parameters. Implicit convertibility from `X` to `Y` should imply implicit convertibility from (immutable) `List<X>` to `List<Y>` and from (immutable) `Map<K, X>` to `Map<K, Y>`. In some cases conversions still could be derived for non-covariant parameters, but, when done implicitly, opportunities for accidental information loss typically arise. Yets such implicit conversions are present in many programming languages including Java and Scala. Let us consider such a case and the problem that it presents.

As we already mentioned, the first parameter (type of keys) in `Map<K, T>` is not covariant. However, if one has a function `f` from type `X` to type `Y`, one could in principle convert a `Map<Y, T>` into a `Map<X, T>` as follows. To obtain a dictionary of type `Map<X, T>` one first applies `f` to a key `x : X` and then uses the result as the key in the original `Map<Y, T>`.

Now consider an instance of `Map<Int, T>` containing three elements:
```
{1 => a;
 0 => b;
-1 => c}
```

An implicit conversion from non-negative integers `Nat` to integers `Int` is perfectly sensible in general. In a language where implicit conversions are passed along the first parameter of `Map<K, T>` it makes the dictionary above implicitly convertable into a `Map<Nat, T>`. However, there is a problem. This new dictionary would contain only two accessible elements:
```
{1 => a;
 0 => b}
```

Here we observe a clear case of information loss. This fact is obscured in Java and similar languages because `Map`s are usually thought of in terms of internal data structures storing their elements, and these are left untouched by the implicit conversion. The implicit conversion above does not erase the entry `-1 => c` from the physical memory, it just makes it inaccessible and invisible as an entry of the `Map<Nat, T>`. When conversions along non-covariant parameters, such as in the example above, take place implicitly, the information loss can easily go unnoticed. Conversions of this kind should only be available as explicit coersions. 

§§ Elaboration ambiguities and overloading issues
-------------------------------------------------

Let us now move on to the next issue and consider the ways implicit conversions might introduce ambiguities into a programming language. The process of transforming implicit conversions into explicit ones is called elaboration. It is usually performed by the compiler internally. There might be cases where more than one conversion path between two types is possible, say `int16 -> int32 -> float64` and `int16 -> float32 -> float64`. In all such cases we must ensure that the results are path independent. When a language with extensible type system allows implicit conversions for custom types (e.g. Scala), this path independence has to be enforced (which cannot be done in Scala).

Further complications arise in languages that make use overloading. By overloading we mean that the same expression may be interpreted differently depending on the types of operands. All possible ways of elaboration still have to produce equivalent results. For example, one might want to use the operator `a / b` both for division of both integers and floats. The result of `a / b` for two integers (say, `int32`) `a` and `b` should remain the same if we were first to convert `a`, `b` or both to `float64`.

That's not the case in C. There, `a / b` means integer division if `a` and `b` are integers, or real division if `a` and `b` are floating point approximations of real numbers. In this example `1 / 2 = 0` if `1` and `2` are interpreted as integer numbers, but `0.5` if they are interpreted as floats. This presents a clear case of incoherent behavior, which can be fixed by using a diffrent operator for integer division (like `//` in Python).

There are, however, more subtle cases. Let us assume `+` is used both for unguarded addition of `int16`s and `int32`s. Adding two large 16 bit integers as `int16`s yields a negative number, whereas adding them as `int32`s would yield a postive number. In case `+` is used for guarded addition of both `int16`s and `int32`s one gets an `#OVERFLOW` in the first case and a proper number in the second one. Thus, overloaded arithmetic operators in low-level languages with implicit conversions  are inevitably incoherent. High-level languages can solve this problem by having an unbounded addition operator `+`. For instance, in Python `+` by default performs addition of fixed length integers, and in case of overflow or underflow it automatically switches to arbitrary precision integer representation as fallback. Yet even in high-level languages, it's extremly involved to combine overloading and implicit conversions without leaving room for incoherencies.

The aforementioned issue of covert information loss can also be seen as a case of elaboration ambiguity. In a language with an overloaded equality operator `x = y`, its coherence with implicit conversions precludes covert information loss. Accidental information loss can be defined as two unequal things becoming equal under an implicit conversion, which whould make two possible elaborations of `x = y` unequivalent.


§§ Subtyping and Inheritance
----------------------------
TODO:
- Subtyping defined by inheritance fundamentally flawed;
- Both subtyping and inheritance clash with mutability;


§ А ещё вот такая проблема ЕСТЬ!
--------------------------------

– Datatype-generic programming not available (impossible to define, say, a `serialize()`-method for generically for all possible classes containing serializable fields only).





§ ПЛАН!!!!111
-------------

Подумать, где и на что хорошо бы сослаться.

После закрытие Digression, сказать: вот мы увидели, что плохо с существующими системами, а сейчас как обещали выше расскажем чего мы по этому поводу хотим сделать. 
Вспомним, что у нас было такие вот три проблемы:
1) Mainstream languages tend to stick with bad typing practices where better ones are available;
2) The gap between statically typed languages and dynamically typed languages has to be closed;
3) There are computational behaviours for which good typing practices are yet to be determined
4) programming techniques for which good typing practices are yet to be determined  
ADD 4 TO THE BEGINNING

Первой уже посвящено много работы, и мы лишь по ходу действия коротко упомянем эти подходы. Второй проблемой мы (пока) непосредственно не занялись, и считаем что о ней пока рано думать. А вот третья  (and 4 ?) проблема это как раз то, над чем мы работаем. 

Дальнейший рассказ основан на том, какие бывают классы типов. 
Их бывают три: data types, purely abstract types, entity types. 
-- Первые очень хорошо поняты (data types), но мы вам про них всё равно расскажем, потому что мы знаем как с ними обращаться хорошо, а этого en masse не делают. Мы покажем как, тем более что для дальнейшего рассказа это надо понимать. (это мэппится на первую проблему которые надо называть словами) 

вторые и третьи позволяют решать четвёртую и третью проблему (которые надо называть словами)

-- purely abstract types А дальше мы расскажем про ещё один класс типов, про него пока мало кто знает как с ним надо, но у нас вроде получилось придумать sound and complete theory.
-- entity types

Есь чо?!

* Вначале есть описание, что типчеги бывают data types, entity types и purely abstract types (извините, это лучшее название которое я выдумал, и это неологизм, потому что имеющиеся в литературе названия вообще непонятнэ: icon types, ghost types, erasable types) с кучей примеров. 
* * Тут надо сказать, что это покрывает все сущности, которые в программировании вообще возникают.

* Есь описание того, как всевозможные data types порождаются двумя очень простыми и базовыми конструкциями.
* * Тут надо показать, что это покрывает не только все иммутабельные типы, которые читатель может себе представить, но и те, которые читатель едва ли ожидал встретить (в частности, “честные” вещественные числа).

* Есть кусок, где я начинаю говорить про то, как устроены эти самые purely abstract types на примере самого важного их экземпляра, и собираюсь углубиться глубоко. Потому как это как раз содержит новизну нашей работы.
* * Это позволяет одновременно две вещи сделать: организовать datatype-generic programming (за отсутствие которого я корю выше Джаву со Скалой) и заниматься математикой прямо внутри кода, беся живых программистов.




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

Type systems supporting abstract parameters (at least of type `*`) are called polymorphic, the ability to handle purely abstract types like `* -> *` and `* -> * -> *` is known as higher-kinded polymorphism, handling `(Nat -> *) -> *` is known as complex kinded polymorphism. If types of further arguments and the return type are allowed to depend not only on abstract parameters, but also on ordinary values, the type system is told to feature dependent types (a part of the name of our research group). 


§ Generating all data types: Synthetic types and Function types
---------------------------------------------------------------

It turns out, that all data types that have found applications in programming and concrete mathematics to this day, including even unbounded arbitrary-precession real numbers, can be defined combining synthetic datatypes (the ones susceptible to pattern matching, see below) and function types `X -> Y`, the types of “mathematical” (i.e. pure, total, deterministic) functions turning values of type `X` into values of type `Y`, or their dependent form where the result type can depend on the input value (denoted by `(x̲ : X) -> Y[x]` or alternatively `∀(x̲ : X), Y[y]`).

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

Futhermore, synthetic datatypes allow parameters to be of type being currently defined as long as recursion stays well-founded:
```
@Data Nat:
  Zero
  Succ(n̲ : Nat)
```

If all constructor parameters of a synthetic type are of a syntethetic type themselves all the way down, the type is called closed synthetic type. In this case all possible values of the synthetic type can be exhaustively enumerated, in particular synthetic types follow the closed-world assumption, all values can be listed and there are no values besides that, equality of two values is decidable by a trivial top-down recursive algorithm. Functions on synthetic types (not necessary closed ones) can be defined by means of structural induction/recursion on the constructors of type (i.e. essentially just pattern matching). Under mild assumptions easily checkable by the compiler they can be shown to be computable and terminating by definition.

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
