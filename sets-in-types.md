Set Theory from a Type-Theoretical Standpoint
=============================================

For almost a century the classical set theory ZFC and its minor extensions were used as the foundation for the whole body of mathematics. The recently emerged Univalent Foundation programme strives to replace ZFC in that role. Univalent foundations depart from the use of classical predicate logic as the underlying formal deduction system, replacing it with an appropriate constructive type theory.

Here we try to discuss the major differences between these approaches and provide a type-theoretical understanding for the classical set theories.

Part 1. Constructive type theories
----------------------------------

Constructive type theories are formal systems dealing with typed values (like numbers, sequences, or matrices): types govern which operations can be applied to values and how. Rather than providing a fixed suite of predefined types, type theories provide machinery to form the required types.

For example, natural numbers ℕ can be defined as the type freely generated by `Zero : ℕ` and `Succ(n : ℕ) : ℕ`. Its values are `Zero`, `Succ(Zero)`, `Succ(Succ(Zero))`, and so on. Types like this, defined “from below” in terms of generators, are called synthetic. Purely synthetic types are exhausted by values that can be manifestly written down: the basic metatheoretical property of constructive type theories justifying the adjective “constructive” is that any term `t` of a synthetic type like ℕ or 𝔹 (a boolean or a bit) is guaranteed to reduce to a finite composition of generators.

This property is also reflected internally: When defining a function or carrying out a proof on a synthetic type, it is allowed to perform case analysis (exhaustive pattern matching) by constructors, i.e. act as if every `n : ℕ` is either `Zero` or `Succ(m)`. This still does not rule out the possibility of infinite chains `Succ(Succ(Succ(...)))` never resolving to `Zero`. However, such function definitions are additionally allowed to be structurally recursive and guaranteed to produce total functions. That rules out the aforementioned infinite chains: if there were any, the functions would never halt. 
As it rather surprisingly turns out, case analysis and structural recursion are sufficient to define not only all arithmetic operations on natural numbers, but in fact, any desired operations on all kinds of synthetic types can be defined this way:
```
is-zero(n : ℕ) := match(n)
  Zero    => 𝟙
  Succ(p) => 𝟘
  

double(n : ℕ) := match(n)
  Zero    => Zero
  Succ(p) => Succ(Succ(p))

plus(n m : ℕ) := match(n)
  Zero    => m
  Succ(p) => Succ(plus(p, m))
```

That's how a simple type definition does not only specify all possible values of that type in a declarative way but also provides the complete toolbox of operations on those values.

Some other types, like the type of functions `f : ℕ -> 𝔹`, are defined as types of “black boxes“ with a certain interface and properties. The type `ℕ -> 𝔹` is the type of “black boxes“ that yield a bit value when given a number `n : ℕ`. Some instances of such functions may be manifestly constructed (namely, the computable functions), but it is not claimed that the type is exhausted by such functions.

The type definition mechanism of constructive type theories can be “abused” to define types that encode relations on other types, for example, the type `LessOrEquals[n m : ℕ]` that is inhabited precisely if `n ⩽ m`. The element of such type is called a witness.

As it turns out, most constructive type theories are rich enough to express first-order logic and deduction: the relations defined like `IsLessThan` can be composed by means of logical junctions and quantifiers. For example, the proposition `∀(n m : ℕ) (n ⩽ m) ‹or› (m ⩽ n)` can be expressed as the type of functions that take arguments `n : ℕ` and `m : ℕ` and produce either a witness of `(n ⩽ m)` or a witness of `(m ⩽ n)`. All proofs not involving double negation elimination `¬¬P -> P` for non-finitary objects (such proofs are known as intuitionistic proofs for historical reasons) can be carried out entirely within the system itself.

When used for proofs, the combination of case analysis and structural recursion is known as mathematical induction, which is available for all synthetic types and implies that non-standard inhabitants of synthetic types (if there were any) would be both non-constructible and non-observable, thus non-existent from the internal standpoint. Basic logics available in most constructive type theories together with induction principles for all synthetic types yield a powerful deduction system that can be used as a basis for mathematical foundations.

All value types available in constructive type theories are either defined synthetically (i.e. “from below” in a constructive fashion)‚ or as black-box types. In the latter case, type theories manifestly refrain from assuming that these types comprise closed totalities known in advance.

This is possible due to the lack of unrestricted double negation elimination `¬¬P -> P` in constructive type theories. For instance, without double negation elimination the Cantorian diagonal argument does not imply that there are strictly more functions `ℕ -> 𝔹` than natural numbers: it only shows that _it is inherently impossible to rule out_ that there are strictly more such functions than natural numbers, and there can be no effective numbering of such functions. It is perfectly compatible with both a Cantorian set-theoretic model, and a model where only computable functions `ℕ -> 𝔹` exist: their set is subcountable, and the lack of effective numbering is due to the halting problem only.

Classical, set-theoretical foundations of mathematics strive to fix all possible properties of the objects they deal with. They were conceived before Gödel's Incompleteness Theorem that implies that no sufficiently rich theory can succeed at it: there always will be objects existence of which is neither implied nor ruled out by axioms of the theory. Constructive type theories embrace this fact rather than trying to patch the holes. The unavailability of unrestricted double negation elimination is not a weakness of constructive type theories, but their feature.

Part 2. Sets in the typed setting
---------------------------------

Sets are collections of mathematical objects without regard for order and multiplicity.

In constructive type theories finite sets `FinSet[T]` can be defined as lists `List[T]` modulo permutations and repetitions. Yet infinite generalizations of lists and finite sets greatly diverge. The standard definition used for sets of elements of some fixed type `T` is essentially the same as predicates `p : T -> Prop` on `T`. The constructor for sets that converts such predicates `p` to sets is called bounded separation `s := { t : T | p(t) }`, and one can get the predicate back from the set by using the element relation `t ∈ s`. One can define subcountable sets as such sets, for which there is a function `f` on ℕ such that no element of the set lies outside of the image of `f`. Without double negation elimination one cannot show that any non-subcountable sets effectively exist.  

Predicative type theories do not host the universal type `Prop`, but only a hierarchy of types `Propⁱ` of increasing complexity. There, one cannot define the type of “all” sets on a type, but only the types of all countable sets or all sets definable on a certain given level of projective hierarchy. 

Since type theories only deal with typed values, one can only talk about “sets of T” for a particular type `T`, not “sets in general”, but material set theories such as ZF(U) and NF(U) study “sets in general” that may contain other “sets in general” (set theories with urelements) or are even required to contain solely other sets (pure set theories). While heterogeneous sets are malformations from a type-theoretical standpoint, pure sets can be understood type-theoretically.

Pure sets satisfy `PSet ≅ Set[PSet]`, but this equation cannot be used as a definition. However, one can define such a type “from below”.
In type theories, there is a notion of universes. Universes are types containing names of other types. It is possible to define a notion of hereditary universes, that is universes containing names of other universes containing names of other universes and so on ad infinitum. Logically consistent type theories cannot have universes containing names of themselves, so we'll never end up with an ultimate hereditary universe but would work with universe hierarchies instead. We'll see that hereditary universes perfectly represent the kind of sets studied by theories of well-founded pure sets such as Z, ZF, and ZFC. Let us begin with a few examples.

Obviously, the empty type 𝟘 is a hereditary universe. For each hereditary universe `U`, the type `Set[U]` of subsets of `U` has a natural structure of a hereditary universe which moreover contains all names for all inhabitants of `U` and for `U` itself. The empty set has only one subset, namely the empty set, thus `Set[𝟘] = {𝟘} ≅ 𝟙` the type with the single element 𝟘 (name of the empty type). `Set[𝟙] = {𝟘, 𝟙} ≅ 𝔹`. `Set[𝔹] = {𝟘, 𝟙, {𝟙}, {𝟘, 𝟙}}`.
```
𝟘 ∈ 𝟙 ∈ 𝔹 ∈ Set[𝔹] ∈ Set[Set[𝔹]] ∈ ···
𝟘 ⊂ 𝟙 ⊂ 𝔹 ⊂ Set[𝔹] ⊂ Set[Set[𝔹]] ⊂ ···
```
At each finite level, we have only finite types so we could have used `FinSet[T]` instead of `Set[T]`. The limit of this hierarchy (the union of all those universes) is a countably infinite type `PureFinSet` of all possible hereditarily finite sets. It is a purely synthetic type “defined from below”. In type theories with the universal type `Prop` of propositions, the above hierarchy (known as von Neumann hierarchy) can be extended beyond `PureFinSet` to the right by sets `PureSetⁱ := Setⁱ[PureFinSet]`. This hierarchy eventually accommodates all sets definable in Zermelo set theory Z. In the generalized sense, these sets are also “built from below“: every set `S` admits only finitely long chains of the form `A ∈ B ∈ ··· ∈ S`. All such chains ultimately end up with an empty set. Yet the way such sets (beyond the set `PureFinSet`) are built is non-constructive. They are non purely synthetic types anymore. Without double negation elimination one cannot show that these powersets are not subcountable.

Hierarchies like the one above are known as cumulative hierarchies: the universe at each level of the hierarchy contains names of all universes on lower levels and all their elements. A cumulative hierarchy is called saturated if universes in the hierarchy only contain names of universes on lower levels. The above hierarchy `𝟘 ⊂ 𝟙 ⊂ 𝔹 ⊂ Set[𝔹]` is not saturated: the type `Set[𝔹] = {𝟘, 𝟙, {𝟙}, 𝔹}` contains the set `{𝟙}` that is not an element of the hierarchy itself. Let us instead consider the hierarchy of all finite types `Fin[n]` interpreted as hereditary universes containing names for types `Fin[m]` for `m < n`, and the type ℕ as the hereditary universe containing the names for all types `Fin[n]`:
```
𝟘 ∈ 𝟙 ∈ 𝔹 ∈ Fin[3] ∈ Fin[4] ∈ ··· ∈ Nat
𝟘 ⊂ 𝟙 ⊂ 𝔹 ⊂ Fin[3] ⊂ Fin[4] ⊂ ···
```
Saturated cumulative hierarchies correspond exactly to the set-theoretic notion of ordinals.

The hierarchy `𝟘 ∈ 𝟙 ∈ 𝔹 ∈ Fin[3] ∈ Fin[4] ∈ ··· ∈ Nat` can be obviously extended to the right: one can define the hereditary universe `ω+1` containing precisely the names for all the `Fin[n]` and the name of ℕ, one can define the universe `ω+2` that additionally contains the name for `ω+1`. The universe with names for all `ω+n` is conventionally called `2ω`, the universe with names for all `2ω+n` is named `3ω`. By providing the names for all `kω + n` we obtain `ω^n`, and then `ω^ω` and `ω^ω^ω`. By providing names for all exponential polynomials in ω we arrive at ε₀, but it is by no means the end. The construction can be extended to accommodate increasingly complex effectively-countable ordinals. As one can see, already countable (hereditarily countable) well-founded sets can bear enormous complexity. One can define a hierarchy (similar to the von Neumann hierarchy) of definable hereditarily subcountable pure sets that eventually contains all such sets.

The von Neumann hierarchy itself
```
𝟘 ⊂ 𝟙 ⊂ 𝔹 ⊂ Set[𝔹] ⊂ Set[Set[𝔹]] ⊂ ··· ⊂ PureFinSet ⊂ PureSet¹ ⊂ PureSet² ⊂ ···
```
can be extended beyond `PureSetⁱ` for finite `i`. Namely, `i` can be taken to be any ordinal by the following definition:
```
PureSetⁱ := ∪ Set[PureSetʲ] for all j ⊊ i
```

The definition does not only refer to constructible (thus effectively-countable) ordinals, but any saturated cummulative hierarchies themselves built from elements of nonconstructive universes PureSetⁱ. It can be shown that all sets of ZF(C) eventually lie in PureSetⁱ for some ordinal `i` definable in ZF(C).