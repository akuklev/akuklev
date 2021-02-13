§ What are dependent types and why do we need them?
===================================================

Here I attempt to explain what basic dependent types to an interested reader who knows enough C to understand the following “Hello, $name!” piece:

```c
main(int argc, char* argv[]) {
  if (argc == 0) {
    printf( "Hello, world!" ); 
  } else {
    printf( "Hello, %s!", argv[0] );
  }
}
```

In C, the procedure being executed automatically when the program is started is called `main`. It has two arguments: the 'argument count' `argc` is the number of command-line arguments and 'argument values' `argv` is the array containing them. The above program prints out "Hello, world!" if executed without command-line arguments or "Hello, {first command-line argument}!" otherwise.


The language is said to support dependent types if one can use values of variables as paremeters of types. In particular, in such a language arguments of a function can be used as parameters of other arguments' types. In a fictional dependent dialect of C, one could have used the following signature for `main(..)` instead:

```c
main(nat argc, string[argc] argv) {
  ...
}
```

Here we assume predefined types `nat` of natural numbers (i.e. non-negative integers), `string` of character strings, and notation `some_type[n]` for arrays of fixed length `n` (where `n` is a `nat`). Thus the signature above states that `argc` is a non-negative integer, and and `argc` is a fixed-length array of character strings, with its length given by `argc`. Now the it is known in compile-time how long `argv` is, so that `index out of bound`-kind errors could be checked in advance.

Here we meet the most basic kind of dependent types: dependent tuples, which are of the form `(X x, Y<x> y,..)`. Here we mean that the first element `x` of the tuple is of type `X`, the second one `y` is of the type `Y` which may depend on `x` as a parameter, as in `(nat argc, string[argc] argv)`. And, of course, tuples with more elements than two are allowed.

* * *

Now let's consider a function taking a non-negative integer `length` as an argument and generating an array of `length` integers:
```
int[] generateRandomSequence(nat length);
```

Here we actually know in advance which length the returned array would have, but there is no way to write it down in C. (This example also shows why types in dependently-typed languages are commonly written after identifiers and not before them.) In the fictional dependent dialect of C we could have written this as follows:
```
func generateRandomSequence(length : nat) : int[length];
```

Here we meet the second basic kind of dependent types: dependent functions.

* * *

Dependency can be more involved. One of the examples would be tagged union. Say, we have a request `open()` to the system which may result either in a filehandle or in an error message:
```
open() : (success : bool, payloadT<success> payload), where
   payloadT<true> := FILEHANDLE
   payloadT<false> := string
```

The more realistic case is a request to a database:
```
db.performQuery(q) : (SqlResultTypeTag tag, payloadT<tag> payload)
```

Consider the function `printf(string fmtstring, ...)`. As the first argument, it accepts a string that may contain one or several placeholders of the form "%[parameter][flags][width][.precision][length]type", say "%f" for `float`. Now if we have a pure total function `printfT<fmtstring : string>` that parses the `fmtstring`, finds out how many placeholders are there and arguments of which types do they require, and generates the respective tuple type, in the fictional dependent C dialect we could have written the exact signature of `printf`:
```
printf(string fmtstring, printfT<fmtstring> ...args)
```

Of course, an SQL query could be parsed equally well:
```
db.performQuery(q, queryArgumentsT<q> ...args) : (SqlResultTypeTag tag, payloadT<tag> payload)
```

If SQL-schema is known in advance, the result type can be also calculated from the query itself:
```
db.performSafeQuery(q, queryArgumentsT<q> ...args) : resultT<q> throws WrongDatabaseSchemaVersion
```

----

Dependent tuple types and dependent function types in this generality are necessary and sufficient to provide exact signatures for public APIs and settled libraries, where exact means:
- the caller will never get an unexpected `InvalidArgumentException` because argument validation can always be performed on the caller side entirely;
- the caller will never have to “cast” the result manually;
- (non-obvious fact that will be elucidated later: ) any mathematically expressable contract relating arguments and result can be expressed as a part of the signature.
