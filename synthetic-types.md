Synthetic Types
===============

The first data types one encounters in general-purpose languages hardware-defined types. Well known examples from C-like languages are the types `int` (`int32`) and `float` (`float64`) of 32-bit integers and 64-bit floating point real numbers respectively. Yet, there are also intrinsically defined types: their definitions and behaviour are independent of any machine-related details.

§ Finite closed types
---------------------

Most simple intrinsically defined types are the enumerations:

**Example 1**
```c
enum State {
  Working,
  Failed
}

enum Digit {
  D0,
  D1,
  D2,
  ..,
  D9
}
```

Here, two enumeration types called `State` and `Digit` respectively are defined. `Working`, `Failed`, `D0` and so on are called constructors of respective types.

Enumerations are finite and closed data types. Finite means that a variable of enumeration data type can attain only a finite set of values. Closedness means that a variable of enumeration data type is guaranteed to be given by a constructor declared in the definition of the enum type. In particular enumeration types cannot be extended ulteriorly and inheritance is forbidden for them.

There are several extensions to enums:
1) Constructors can be allowed to have parameters;
2) Enumeration types can be allowed to have additional equalities.

The following example, written in a C-like pseudocode, shows both extensions:

**Example 2**
```c
enum Color {
  Red,
  Green,
  Blue,
  CustomColor(Digit r, Digit g, Digit b),
  
  // Equalities:
  Red   => CustomColor(D9, D0, D0),
  Green => CustomColor(D0, D9, D0),
  Blue  => CustomColor(D0, D0, D9)
}
```

This extensions do not spoil closedness and finiteness as long as all parameters are also given by closed finite enums.

§ Closed inductive types
------------------------

One can go beyond finitness by allowing parameters of the constructors to have the type being defined. Let's consider the most basic example:

**Example 3**
```c
inductive NaturalNumber {
  Zero,
  SuccessorOf(NaturalNumber n)
}
```

The possible values of a variable of type `NaturalNumber` are `Zero`, `SuccessorOf(Zero)`, `SuccessorOf(SuccessorOf(Zero))`, etc. Such types are called closed inductive data types. They are not finite anymore, but are effectively enumberable: for each closed inductive data type one can explicitly write down a program that prints a sequence of all its possible values.

Inductive types are inherently immutable and are not allowed to contain any cycles. A number `inf = SuccessorOf(inf)` is not allowed (cycles are forbidden) and cannot be constructed because the parameters of constructors are required to be given by already defined immutable values of respective types.

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

Any possible value of a variable is given by a tree of constructors of finite depth. A value of an inductive type can be thus analysed by recursive pattern matching and the recursion is guaranteed to terminate (no cycles => no infinite loops).

Inductive types are also compatible with custom equalities:

**Example 5**
```c
inductive Integer {
  Positive(NaturalNumber n),
  Negative(NaturalNumber n),
  IntegerZero
  
  // Equalities:
  Positive(Zero) => IntegerZero
  Negative(Zero) => IntegerZero
}

inductive UnorderedPairOfIntegers {
  UPair(Integer a, Integer b)
  
  // Undirected equalities:
  (Integer A, Integer B) => UPair(a, b) = UPair(b, a)
}
```

If all equalites in a closed inductive type are directed, the equality of two values is decidable, i.e. equality can be checked by an algorithm which is guaranteed to terminate. If some of the equalities ere not directed (see UnorderedPair in the Example 5), the equality checking is only guaranteed to be semidecidable, i.e. to terminate if the values are equal indeed. If the values are distinct, equality checking might run into an infinite loop. That is not a flaw of a particular equality checking algorithm, but an general problem known in mathematics as [word problem undecidability](https://en.wikipedia.org/wiki/Word_problem_(mathematics)).

§ More General Closed Synthetic Types
-------------------------------------

Inductive-inductives, iir (dependent types + calculation)

§ Non-Closed Synthetic Types
----------------------------
