What are dependent types?
=========================

I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). This article is an attempt to introduce _dependent types_ for an interested reader who knows enough [C](https://en.wikipedia.org/wiki/C_(programming_language)) to understand the following “Hello, world!” piece:

```c
main(int argc, char* argv[]) {
  if (argc == 0) {
    printf( "Hello, world!" ); 
  } else {
    printf( "Hello, %s!", argv[0] );
  }
}
```
It prints out `"Hello, world!"` if executed without command-line parameters or `"Hello, {first command-line argument}!"` otherwise.

From a more general perspective, each C program has a unique function called `main()`. When a program is executed, it is precisely the `main()` function which is being called. `main()` has two arguments:
* `argc`: 'argument count' is the number of command-line arguments; 
* `argv`: 'argument values' is the array containing them.

The argument `argc` is declared as an integer (`int`) and tacitly assumed to be non-negative. The signature `char* argv[]` is an archaic way to declare that `argv` is an array of unspecified length of strings. The length of `argv` is tacitly assumed to be `argc`.

There is a problem with such tacit assumptions: they can be easily violated; mostly by mistake, but sometimes also maliciously. Failure of tacit assumptions is responsible for myriads of crashes and security breaches. In fact, the vast majority of security vulnerabilities are of that kind. Running the example above with `argc = 1` while the true length of `argv` is zero would result in either printing out gibberish of the form `Hello, %$Gz#H@...` of humongous length or in a segmentation fault (system crash).

The solution is to make tacit assumptions explicit. It goes beyond the capabilities of C, so we will have to recourse to C-like pseudocode:
```cpp
main(nat argc, string[argc] argv) {
  ...
}
```

This signature is meant to state that `argc` is a natural number (= non-negative integer), and `argv` a fixed-length array of strings with its length given by `argc`. The real C used to support fixed-length arrays, like `int arr[3]` for an array of three integers, but the expression in square brackets had to be a compile-time constant. However, in order to state what we actually know about length of `argv` we have to allow expressions, values of which are not determined in compile-time. In other words, we have to allow types to depend on "run-time" values — that's where the name “dependent types” comes from. Some programming languages, unlike C, support such signatures, or, more formally:

<dl><dt>Definition</dt>
  <dd>A programming language is said to support dependent types if it allows one or several arguments of a function to be used to specify the types of the following arguments or the return type.</dd>
</dl>

In dependent languages, return types of functions are always written not at the beginning of a declaration (like in `int n`), but at its end. For example, a declaration of a function returning an integer looks as follows: `get_count() : int`. That's because return types are also allowed to depend on the arguments:
```c
generate_random_sequence(nat length) : int[length];
```

§ Type-level programming
------------------------

Now let's turn our attention to the function 'print formatted' `printf(string template, ...)`. In the example above, it was used to print “Hello, world!” and “Hello, {name}!”:
```c
printf( "Hello, world!" ); 
...
printf( "Hello, %s!", argv[0] );
```

This function has a variable number of arguments depending on the first argument `template`. If `template` contains no %-patterns, `printf` has no additional arguments. If it has a single `%s`, as in our example, it has an additional argument of type `string`. The pattern `%d` would require an integer argument, and `%f` a `float`.

In a sufficiently powerful dependently typed language, we can write down a precise signature for such a function:
```c
printf(string template, printf_args<template> ...args)
```

where `printf_args<template>` is a “type-valued function” that parses the `template` and returns the list of types of the required additional arguments. In our example
```cpp
printf_args<"Hello, %s! Current CPU temperature is %f.">
```
would return `(string, float)`.

Precise signatures like this are desirable for public APIs and settled libraries so that argument validation can be performed beforehand (for public APIs) and in compile-time (for settled libraries). Type-level functions are a part of the signature and must be executable in compile time and/or on a remote machine. Therefore, they have to return a result for all inputs employing no side effects (no input/output, no exception throwing etc). In “sufficiently powerful” languages all manifestly terminating side-effect free functions can be cast to type level. In such languages **any restrictions on the arguments and any contracts relating the arguments and the result can be expressed as a part of the signature**.   
{**TODO:** Про контракты нипанятна, особенно contracts relating the arguments and the result}

For software developers who have experience writing database-facing code, let me mention a use case of profound importance.  
Given the database schema is known in advance, the types of arguments and results for a given query can be figured out by parsing the query. Since importing of the database schemata can be integrated into the build process, the following Java'esque signature would be possible:
```Kotlin
db.performQuery(string q, db.query_args<q> ...args) : db.query_result<q> throws IncompatibleDbSchemaException
// The `IncompatibleDbSchema` exception arises if the database schema has
// changed since the application was compiled, so it has to be rebuilt.
```

{**TODO Завершающий абзац:** Закруглить, что последний пример показывает какое у завтипов офигенное прямое практическое приминение, и сказать что their scope goes far beyond this. И вернуться к уровню статьи, что “мы строили-строили и наконец построили”.}
