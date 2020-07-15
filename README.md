My profile
==========

Research interests:
<details>
<summary>Languages for formalized mathematics.</summary>
  
Requirements:

1. Nice handling of constructive Concrete Mathematics, Real Analysis, Basic Linear, Commutative and Universal Algebra, (non-higher) Category Theory
2. Support LEM and set-level AC as modalities. Ideally, also weaker forms of AC, nameley Dependent Choice and Ultrafilter Lemma.
3. (ideally) Eat itself (“The Gentle Art of Levitation” kind) and support Higher Category Theory
 
Technically, (1) requires:

1. Enough Univalent Universes and QIITs, in particular:
     * Unordered collections (unordered tuples, lists, finite sets, etc.)
     * Cauchy Reals and Partiality Monad
     * FOLDS-models satisfying higher structure identity principle
2. Have or fake subset types, support algebraic ornaments with implicit conversions
3. Have proper “large” categories and ZMC/S-style handling of smallness, enough commulativity to prove a nice version of Yoneda
  
  We don't know yet, how to implement (3), but having enough quotient-inductive-inductive-recursive types to have initial algebras for all XATs is certainly a start. Perhaps having Equaliser Coinductive Types (dual of QITs) containing a form of strict equality in disguise, should be enough to define higher functors.
</details>

<details>
<summary>Extended Algebraic Theories: Bidirectionally presented type theories as algebraic presentations of higher categorcal stuctures.</summary>
* Intuitionistic type theory with liberal identity type (i.e. without uniqueness of identity proofs) and dependent sums is known to play a role of “algebraic” presentation of finitely complete ∞-categories, if we add dependent products locally cartesian closed ∞-categories arise, addition of univalent universes leads to ∞-pretoposes. Type-theoretic description of weak ω-categories was recently given.
* We elaborate this intuition to a notion of extended algebraic theories (XATs), which can be seen as a generalization of Cartmell's Generalised Algebraic Theories w/o equations on sorts, and as stratification of Essentially Algebraic Theories. In particular, the initial model existence proof for EATs can be adopted. 
* We describe their categorical (“functorial”) semantics in the doctrine of weak model categories.
* We show how bidirectionally presented type theories are XATs.
* We study two instances of Baez-Dolan microcosm principle:
  * Category of models of a given theory is a weak model category itself
  * For a large class of theories `T`, a theory `T'` can be derived, so that there is a natural notion of `V`-enriched `T`-s for each `T'`-algebra `V`. For example, for the theory `T` of ordinary categories, `T'` is the theory of virtual double categories, so that for any virtual double category `V` the notion of `V`-enriched categories can be defined.
</details>
<details>
<summary>Algebraic Theories over a Generalised Field `K`, including the “Field with one element” 𝔽₁</summary>
  
Develop a syntactic formalism and a doctrine implementing semantics for Algebraic Theories over a Generalized Field `K`. [In the case of 𝔽₁, the logic of joinable partial computations should emerge.](https://github.com/akuklev/algebraic-theories/blob/master/K-algebraic-theories.md), Hopf Algebras reduce to Groups etc. For complex disk, the theories relevant to quantum measurement and entanglement might probably emerge.
</details>
<details>
<summary>Modal Type Theory, Computational Effects</summary>
  
* Adjoint Logics to encompass concurrency (typed actor model)
* Linear Type Theory (Spectra, Quantum Computing)
* Typed quantum actor model?
* Algebraic effects as finitely-presented monads: free monads modulo “relations” in form of a related Dijkstra monad
* Algebraic coeffects: as protocols of communication to indeterministic objects with incapsulated mutable state
</details>

<details>
<summary>Diophantine Reducibility Logic: A truly finitist logical system for ordinal analysis</summary>
  
* Elaborate on duality between rapidly-growing “Nat -> Nat” functions and natural ordinal notation systems (a form of inductive types), provide a straightforward way to encode both by a diaphantine equation: equation has solution for arbitrary `x` ⇔ “the relational description does indeed define a total rapidly-growing function” ⇔ “transfinite induction up to `θ` always terminates”.  
* A finitist logical system for ordinal analysis (where complex cut elimination proofs can be carried out) implemented as extrinsic type system for diophantine equations. No unconditionally true judgements beyond ω², conditional judgements of the form “well-founded induction up to `θ` proves consistency of logical system `Ξ`” with meaning explanation “totality of a relationally-described rapidly-growing function for arguments up to `n` implies cut-elimination for Ξ-sentences with length up to `n`”. Ideally a self-verifying theory.
</details>
   * 🔭 [The type-theoretic cousin of ZMC/S: A particularly pleasing foundation for category theory](https://github.com/akuklev/QIITs-in-Cedille).

Applied interests:
<!--
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...

-->
