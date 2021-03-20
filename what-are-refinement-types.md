Refinement types and Witness types
==================================

The claim that dependent types are sufficient to enforce argument validation of any desired complexity actually requires (mild) additional assumptions. So let's dive into the details if you're interested.

In the first section we used the type `nat` of non-negative integers, which can be understood as restriction of integers by a predicate `n >= 0`. Such types are called refinement types:

<dl><dt>Definition 2</dt>
  <dd><i>Refinement type</i> is a type restricted by a (semidecidable) predicate which is assumed to hold for any element of the refined type.</dd>
</dl>

Examples:
```
nat := {int n | n >= 0}
SortedList<T> := {List<T> list | isSorted(list)}
```

Inhabitants of `SortedList<T>` are thus precisely such lists `List<T>`, that `isSorted(l)` returns `true`. The word `semidecidable` in the Def. 2 refers to the requirement that the predicate is given by a “checking” algorithm. The algorithm is not required to terminate on every input (if it does, the predicate is called decidable). 

With refinement types one can express restrictions on arguments (as we have already done with `argc` by using `nat` instead of `int` as its type) or postconditions when used for return types:
```
sort(List<T> list) : SortedList<T>
```

Finding a name for every refinement type is a bit tedious and bloats the code, so let's introduce a lightweight inline notation:
```
f(int i, int j, {i < j})
```

This funcion is supposed to accept an arbitrary inter `i` and an integer `j` greater than `i`. Yet this notation can be also read as
```
f(int i, int j, {i < j} invisible_argument)
```
where `{i < j}` is the type of witnesses certifying that `i < j` returns `true`. It is, `f` is now a function with tree arguments: two arbitrary integers and a witness that the second one is greater than the first one.

<dl><dt>Definition 3</dt>
  <dd><i>Virtual witness type</i> for a given proposition is a virtual type assumed to be inhabited by certificates that the proposition holds. By "virtual" we mean that arguments and variables of those types are used only in compile-time (for type-checking, termination checking and instantiation) and are so to say erased in the compilation process. Certificates do not "exist" in the run-time, they do not have any representation in terms of real bits and bytes in memory.</dd>
</dl>

In particular, virtual witness types `{expr}` are inhabited if and only if `expr` evaluates to `true`. In presense of witness types, refinement types can be considered a special case of dependent types. Witness types are either present or can be emulated in all dependently-typed languaged known to the authors.

