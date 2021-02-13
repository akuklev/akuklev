What are dependent types?
=========================

I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/).

This article is an attempt to explain dependent types to an interested reader who knows enough [C](https://en.wikipedia.org/wiki/C_(programming_language)) to understand the following “Hello, world!” piece:

```c
main(int argc, char* argv[]) {
  if (argc == 0) {
    printf( "Hello, world!" ); 
  } else {
    printf( "Hello, %s!", argv[0] );
  }
}
```

Each C program has a unique procedure called `main()`. When a program is executed, it is precisely the `main()` which is being called. It has two arguments: the 'argument count' `argc` is the (`int`eger) number of command-line arguments and 'argument values' `argv` is the array containing them. The signature of `char* argv[]` is an archaic way to write down that `argv` is an array of character strings of unspecified length. The above program prints out "Hello, world!" if executed without command-line arguments or "Hello, {first command-line argument}!" otherwise.

Working definition
: Language is said to support dependent types iff arguments of a function can be used as parameters of other arguments' types.

In a fictional dependent dialect of C, one could have used the following signature for `main(..)` instead:
```c
main(nat argc, string[argc] argv) {
  ...
}
```

Here we assume predefined types `nat` of natural numbers (i.e. non-negative integers), `string` of character strings, and notation `some_type[n]` for arrays of fixed length `n` (where `n` is a `nat`ural number). Thus the signature above states that `argc` is a non-negative integer, and and `argc` is a fixed-length array of character strings, with its length given by `argc`. Now the it is known in compile-time how long `argv` is, so that `index out of bound`-kind errors could be checked in advance.


§ Type-level functions
----------------------

Now let's turn our attention to the function 'print formated' `printf(string fmtstring, ...)`. It has a variable number of arguments depending on the first argument `fmtstring`. If `fmtstring` contains no %-patterns, `printf` has no additional arguments. If it has a single `%s`, as in our example, it has an additional argument of type `string`. The pattern `%d` would require an integer argument, and `%f` a float. To make the signagture of `printf` precise we need to write a type-level function `printfT<fmtstring>` that parses `fmtstring` and returns the respective tuple type: `printfT<"Hello, %s! Current CPU temperature is %f."> == (string, float)`. With such a function one could write the signature of `printf` as follows:

```c
printf(string fmtstring, printfT<fmtstring> ...args)
```

Exact signatures like this are desirable for public APIs and settled libraries so that argument validation can be performed beforehand (public APIs) and in compile-time (settled libraries), thus type-level functions are a part of the signature and should be executable in compile time/on a remote machine. Thus, they have to be pure and manifestly terminating.

Let me write down a very simple example of a type-level function just to give an impression. Let's assume we have a function `send(text_or_data, payload)`, where the type of payload is either `string` or `byte[]` depending on the value of the first argument:
```c
send(bool text_or_data, payloadT<text_or_data> payload), where
   type payloadT<true> := string;
   type payloadT<false> := byte[];
```

In good languages supporting dependent types, any pure and manifestly terminating function can be lifted to the type level.



db.performQuery(q, queryArgumentsT<q> ...args)
  


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
