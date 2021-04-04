Let `a b : X` be two inhabitants of some type `X`. Let us denote the type of identifications between those inhabitants by `a = b`.
An identification `e : (a = b)` witnesses that `a` and `b` are equivalent and provides a way to turn any constructions and any proofs involving `a` into
constructions and proofs involving `b`. That is we can “apply” functions `f(x : X) : Y` not only to elements `x` of `X` but also to eqialities. Applying `f`
to the equality `e : (a = b)` would yield the equality `f(a) = f(b)`. This should also work for dependent functions (which are used for proofs). That is,
`p(x : X) : Y(x)` can be also applied to `e : (a = b)` yielding `p(e) : (p(x) =[e]= p(b))`.

Now let us define the following type:
```scala
structure UniformBrigde<T : *> {
  origin : T
  terminus : T
  connection : (origin = terminus)
}
```

We'd gain a lot of syntactical comfort if we were able to formulate this type as a function type: `c : II -> T` with
```
c(Origin) : T, 
c(Terminus) : T,
c(Connection) : f(Origin) = f(Terminus)
```

With such type application of functions to identifications is just a function composition. Let
```
c(Origin) := a
c(Terminus) := b
c(Connection) := e
```

Then
```
(f.c)(Origin) = f(c(Origin)) = f(a)
(f.c)(Terminus) = f(c(Terminus)) = f(b)
(f.c)(Connection) =def= f^(e) : (f(a) = f(b))
```

For dependent functions the composition would be a dependent uniform bridge `c : ∀(i : II), T(i)`, which is the following structure in disguise:
```
structure DepUBridge<T : II -> *> {
  origin : T(Origin)
  terminus : T(Terminus)
  connection : (origin =[T]= terminus)
}
```

* * *

Now let us try to understand the nature of identification types a bit better.

Consider a function
```
f(x : X, y : Y) : Z
```

We may require that it depends on one or more of its inputs _uniformly_ this way:
```
f(x : X, y : 0Y) : Z
```

Uniform dependence means that the return value does not repend on this argument, only the return type. While for the particular type, there are lots of functions `T -> T` there is only one that can be defined uniformly for all types, the identity. Uniform dependence forms an intersection of them: `f : ∀(T : *), T -> T` does only contain the identity function, this type contains the bare minimum: the "greatest common denominator" or all types of such form. 

Functions can only depend on their kind arguments uniformly, because elements of kinds are types which cannot be used in the body of the function in any way. But this does not apply to type formers.

Type formers depend on their type parameters non-uniformly, they use them to generate a return value, namely the type. When we generate type formers manifestly, by taking a function for universe `f(t : U) : U` and lifting it to `f* : * -> *`, we see it would not work if the function would have used universe uniformly `f(t : 0U) : *`.

Can there be a type former depending on its type arguments uniformly?
Consider the type former with signature `I(T : 0*) : T -> T -> *`. For type `T` it produces a type dependent on pairs `(x y : T)`, i.e. a proof-relevant relation on the type `T`. It should be a relation uniformly definable for all types `T`. The relation uniformly definable for all types should be precisely the equality on this type.

Now we have only written down the signature for `I`. Can we also write down its body?
```
// только нарушая, используя T в теле:
f(T : 0U) := \(x y : T), {II -> T with lt => x, rt => y}
```

```
structure Bridge<T : *> {
  lt : T
  rt : T
  conn : T -> T -> *
}

structure PolyBridge<Bridge<*>> {
}
```

Релейшины — это просто I -> (ϰ) не для типов, а для кайндов.
