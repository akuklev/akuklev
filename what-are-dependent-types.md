What are dependent types?
=========================

I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). This article is an attempt to explain dependent types to an interested reader who knows enough [C](https://en.wikipedia.org/wiki/C_(programming_language)) to understand the following “Hello, world!” piece:

```c
main(int argc, char* argv[]) {
  if (argc == 0) {
    printf( "Hello, world!" ); 
  } else {
    printf( "Hello, %s!", argv[0] );
  }
}
```

Each C program has a unique procedure called `main()`. When a program is executed, it is precisely the `main()` procedure which is being called. It has two arguments:
* `argc`: 'argument count' is the number of command-line arguments; 
* `argv`: 'argument values' is the array containing them.

The signature `char* argv[]` is an archaic way to say that `argv` is an array of unspecified length of character strings. The above program prints out `"Hello, world!"` if executed without command-line arguments or `"Hello, {first command-line argument}!"` otherwise.

<dl><dt>Definition</dt>
  <dd>A programming language is said to be dependently typed if it allows one or several arguments of a function to be used to specify the types of the following arguments or the return type.</dd>
</dl>

In a fictional dependent dialect of C, one could have used the following signature for `main(..)` instead:
```cpp
main(nat argc, string[argc] argv) {
  ...
}
```

Here we assume predefined the types `nat` of natural numbers (= non-negative integers) and `string` of character strings; and the notation `element_type[n]` for arrays of fixed length `n` (where `n` is a non-negative integer, and `element_type` some type). Thus the signature above states that `argc` is a non-negative integer, and `argv` is a fixed-length array of character strings, with its length given by `argc`. This signature declares how long `argv` is, so that `index out of bound`-kind errors could be checked during compile time.

In dependent languages, types are usually written not at the beginning of a declaration (like in `int n`), but at its end at least for functions. For example, in Typescript, Scala, Kotlin, F#, Agda, etc. a declaration of a function returning an integer looks as follows: `get_count() : int`.  
There is a good reason for it: return types are also allowed to depend on the arguments.

```c
generate_random_sequence(nat length) : int[length];
```

With dependent typing one can avoid manual casts (coercing values into the “right” type) altogether.

§ Type-level programming
------------------------

Now let's turn our attention to the function 'print formatted' `printf(string template, ...)`. In the example above, it was used to print “Hello, world!” and “Hello, {name}!”:
```
printf( "Hello, world!" ); 
...
printf( "Hello, %s!", argv[0] );
```

It has a variable number of arguments depending on the first argument `template`. If `template` contains no %-patterns, `printf` has no additional arguments. If it has a single `%s`, as in our example, it has an additional argument of type `string`. The pattern `%d` would require an integer argument, and `%f` a `float`. Can we use dependent typing to write down its precise signature?

In a sufficiently powerful dependently typed language, we can do this:
```c
printf(string fmtstring, printf_args<fmtstring> ...args)
```

where `printf_args<fmtstring>` is a “type-valued function” that parses `fmtstring` and returns the respective tuple type:
```cpp
printf_args<"Hello, %s! Current CPU temperature is %f.">
===
struct {string arg1, float arg2}
```

Exact signatures like this are desirable for public APIs and settled libraries so that argument validation can be performed beforehand (public APIs) and in compile-time (settled libraries), thus type-level functions are a part of the signature and must be executable in compile time/on a remote machine. Thus, they have to be manifestly terminating and employ no side effects (no IO, no exception throwing etc). In “sufficiently powerful” languages all manifestly terminating side-effect free functions can be lifted to type level. In such languages **any restrictions on arguments and any contracts relating arguments and result can be expressed as a part of the signature**.

For software developers who have experience writing database-facing code, let me mention a use case of profound importance.  
Given database schema is known in advance, types of arguments and results for a given query can be figured out by parsing the query. Since importing of the database schemata can be integrated into the build process, the following signature (here we use a more Java'esque syntax) would be possible:
```C#
db.performQuery(string q, db.query_args<q> ...args) : db.query_result<q> @throws IncompatibleDbSchemaException
```
(the `IncompatibleDbSchema` exception being thrown if the schema of the database changed in the meantime, so application has to be rebuilt).
