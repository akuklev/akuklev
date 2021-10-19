Classical modality for constructive type theories
=================================================



1. The setup
------------

Assume we work in an intentsional Martin-Löf Type Theory with empty type `𝟘`, unit type `𝟙 := {𝟘}`, boolean (or bit) type `𝔹 = {𝟘, 𝟙}`, a universe of manifestly propositional types `SProp ⊃ 𝔹` and above it, an infinite cumulative hierarchy of univalent universes `SProp ⊂ 𝒰 : 𝒰⁺ : 𝒰⁺² : ···`, closed under dependent products `∀(X : *) (Y : X -> *)`, dependent sums `Σ(X : *) (Y : X -> *)`, identity types `Id[T : *](x y : T)` and quotient inductive-inductive types including propositional truncation `∃(T : *)`, that allows defining logical disjunction `A ∨ B := ∃(A ⊕ B)` and effective existential quantifier `∃(X : *) (P : X -> *) := ∃(Σ X Y)`.

We'll also assume propositional resizing in the form of an axiom that for each propositional type in any universe there is an isomorphic type in `SProp` and hence in every universe, where
```
(T : *) is propositional := ∀(x y : T) x = y
```
Under univalence, the property of being propositional is itself propositional. Propositional truncation makes a propositional type from any other, but it doesn't make them manifestly propositional (i.e. belong to `SProp`), merely isomorphic to an unknown type from `SProp`.

For convenience, we'll also define the quantifier of effective unique existence
```
(T : *) is singleton := Σ(x : T) ∀(y : T) x = y

∃!(X : *) (P : X -> *) := (Σ X P) is singleton
```
and 
```
(T : *) is plain := ∀(x y : T) (x = y) is propositional
```
The properties of being singleton and of being a plain type are propositional as well. All quotient inductive-inductive types are plain as well as dependent products and sums between them. Universes can however be shown not to be plain under univalence.

2. Non-effective existence
--------------------------

Now let us introduce the quantifier of non-effective existence `∃⁰(X : *) P : X -> *`.

Non-effectively existing sequence `∃⁰(s : ℕ -> ℕ) P(s)` cannot be applied to a natural number `n : ℕ`. Non-effectively existing natural number `∃⁰(n : ℕ) P(n)` cannot be matched against `Zero` and `Succ`. It is, however, possible to apply constructors or leave the value as is. For example, if we know that `∀(n : Nat) P(n) => Q(Succ(n))`, we can use `∃⁰(n : ℕ) P(n)` to prove `∃⁰(m : ℕ) Q(m)`. ∃⁰ can be seen as an operator turning every type into a type with the same constructors but now eliminators.

Additionally, we want to allow to use non-effectively existing values as type parameters for types `T(p : P)` iff `P : SProp`. These two kinds of usage do not compromise evaluation of closed terms and decidability of conversion despite non-computability of non-effectively existing values.

Now that we have tentatively assured this metatheoretical property, we want to postulate the premises of the form `∃⁰(x : X) P(x)` to be satisfiable by `¬∀(x : X) ¬P(x)`. In particular, it also implies that for propositions `P` the premises of the form `∃⁰P := ∃⁰P, 𝟙` are satisfiable by `¬¬P`.

Now let us observe that ∃⁰ is the “classical modality“ under which unrestricted classical reasoning including choice is available without compromising the computational properties and constructive nature of the underlying type theory.

(TBC)
