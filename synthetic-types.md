Synthetic Types
===============

The first data types one encounters in general-purpose languages hardware-defined types. Well known examples from C-like languages are the types `int` (`int32`) and `float` (`float64`) of 32-bit integers and 64-bit floating point real numbers respectively. Yet, there are also intrinsically defined types: their definitions and behaviour are independent of any machine-related details.

§ Finite closed types
---------------------

Most simple intrinsically defined types are the enumerations:

**Example 1**
```
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

Enums are finite and closed data types. Finite means that a variable of enumeration data type can attain only a finite set of values. Closedness means that a variable of enumeration data type is guaranteed to have one of the named values declared in the definition of the enum type. In particular enumeration types cannot be extended ulteriorly and inheritance is forbidden for them.

There are several extensions to enums:
1) They can be allowed to have parameters;
2) They can be allowed to have additional equalities.

The following example, written in a C-like pseudocode, shows both extensions:
**Example 2**
```
enum Color {
   Red,
   Green,
   Blue,
   CustomColor(Digit r, Digit g, Digit b),
   // Equalities
   Red   => CustomColor(D9, D0, D0)
   Green => CustomColor(D0, D9, D0)
   Blue  => CustomColor(D0, D0, D9)
}
```

This extensions do not disturb closedness and finiteness as long as all parameters are also given by closed finite enums.

§ Closed inductive types
------------------------
