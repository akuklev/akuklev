

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
