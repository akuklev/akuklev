Research interests:
- Languages for formalized mathematics:
   * 🔭 [The type-theoretic cousin of ZMC/S: A particularly pleasing foundation for category theory](https://github.com/akuklev/QIITs-in-Cedille).
   * Striving to expand it to meet a the the following requirements/ 
  <details>
  <summary>Requirements</summary>
 
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
- XATs
- Algebraic Theories over a Generalised Field:
    * Develop a syntactic formalism and a doctrine implementing semantics for Algebraic Theories over a Generalized Field `K` which may be a classical field but also the 𝔽₁ (“Field with one element”), reducing Hopf Algebras to Groups, etc. [In the case of 𝔽₁, the logic of joinable partial computations should emerge.](https://github.com/akuklev/algebraic-theories/blob/master/K-algebraic-theories.md)
- Modal Type Theory, Computational Effects:
    * Adjoint Logics to encompass concurrency (typed actor model)
    * Linear Type Theory (Spectra, Quantum Computing)
    * Typed quantum actor model?
    * Algebraic effects as finitely-presented monads: free monads modulo “relations” in form of a related Dijkstra monad
    * Algebraic coeffects: as protocols of communication to indeterministic objects with incapsulated mutable state
- Diophantine Reducibility Logic:
    * Elaborate on duality between rapidly-growing “Nat -> Nat” functions and natural ordinal notation systems (a form of inductive types), provide a straightforward way to encode both by a diaphantine equation: equation has solution for arbitrary `x` ⇔ “the relational description does indeed define a total rapidly-growing function” ⇔ “transfinite induction up to `θ` always terminates”.  
    * A finitist logical system for ordinal analysis (where complex cut elimination proofs can be carried out) implemented as extrinsic type system for diophantine equations. No unconditionally true judgements beyond ω², conditional judgements of the form “well-founded induction up to `θ` proves consistency of logical system `Ξ`” with meaning explanation “totality of a relationally-described rapidly-growing function for arguments up to `n` implies cut-elimination for Ξ-sentences with length up to `n`”. Ideally a self-verifying theory.

Applied interests:
<!--
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...

-->
