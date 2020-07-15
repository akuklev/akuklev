## Current activity
🔭 (WIP) The type-theoretic cousin of ZMC/S: A particularly pleasing foundation for category theory: http://github.com/akuklev/QIITs-in-Cedille.

👯 I’m looking to collaborate on http://github.com/akuklev/algebraic-theories.

## Research interests
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
<summary>Algebraic Theories over a Generalised Field <code>K</code> (including the “Field with one element” 𝔽₁)</summary>
  
Develop a syntactic formalism and a doctrine implementing semantics for algebraic theories over a generalized field `K`. [In the case of 𝔽₁, the logic of joinable partial computations should emerge.](https://github.com/akuklev/algebraic-theories/blob/master/K-algebraic-theories.md), Hopf Algebras reduce to Groups etc. For complex disk, the theories relevant to quantum measurement and entanglement might probably emerge.
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

## Applied interests

<details>
<summary>Syntax for the Working Mathematician: Once the workable theoretical basis for formalized math is available, nice syntax and proper tooling will be urgently needed. We develop a carefull combination of strengths of several math-oriented languages (Fortress, Agda, Coq, Lean, Isabelle, Julia and Python) perfected by through UX analysis: <a href="https://github.com/akuklev/akuklev/blob/master/Sketch_for_a_Common_Syntax%20(1).pdf">view pdf</a></summary>

</details>

<details>
<summary>A Geometric Proof Language (mainly, for planar eucledian geometry as taught in schools), Gamified School Geometry</summary>
  
School Geometry is the subject where most students first encounter formal proofs. A language for machine-verifiable yet easily human-readable proofs would greatly expand accessability of good math education, especially if incorporated into a visial tool (an app for tablet computers) that helps to construct one's own proofs, understand the other's proofs, play, share and learn the language and tools step-by-step in a gamified fasion. <www.euclidea.xyz>
</details>

<details>
<summary>Gamification of skill training for school and college-level mathematics (precalculus and linear algebra).</summary>
  
Doing hundreeds of excersises to learn how to calculate, how to deal with algebraic expressions, solve equations, etc. is extremly boring, while being the only known way to internalise the language of mathematics. At the age of clip culture-leaning progressive education and ever increasing ADHS prevalence, most kids never get fluent enough in quantitative and conceptual languages of mathematics and natural sciences to enjoy them. Gamification seems to be at least a partial way out. <www.triradix.com>
</details>


<details>
<summary>Purified and Improved replacement for HTML+CSS+JavaScript for clean, accessible and consistent interactive documents and FRP-based GUIs: http://akuklev.github.io/html-reworked</summary>
</details>
