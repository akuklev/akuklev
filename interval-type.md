

Let `a b : X` be two inhabitants of some type `X`. Let us denote the type of identifications between those inhabitants by `a = b`.
An identification `e : (a = b)` witnesses that `a` and `b` are equivalent and provides a way to turn any constructions and any proofs involving `a` into
constructions and proofs involving `b`. That is we can “apply” functions `f(x : X) : Y` not only to elements `x` of `X` but also to eqialities. Applying `f`
to the equality `e : (a = b)` would yield the equality `f(a) = f(b)`. This should also work for dependent functions (which are used for proofs). That is,
`p(x : X) : Y(x)` can be also applied to `e : (a = b)` yielding `p(e) : (p(x) =[e]= p(b))`.

