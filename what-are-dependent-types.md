What are dependent types?
=========================

I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). This article is an attempt to explain what are _dependent types_ all about to an interested reader who knows enough [C](https://en.wikipedia.org/wiki/C_(programming_language)) to understand the following “Hello, world!” piece:

```c
main(int argc, char* argv[]) {
  if (argc == 0) {
    printf( "Hello, world!" ); 
  } else {
    printf( "Hello, %s!", argv[0] );
  }
}
```

This example program prints out `"Hello, world!"` if executed without command-line arguments or `"Hello, {first command-line argument}!"` otherwise.
Each C program has a unique function called `main()`. When a program is executed, it is precisely the `main()` function which is being called. It has two arguments:
* `argc`: 'argument count' is the number of command-line arguments; 
* `argv`: 'argument values' is the array containing them.

The argument `argc` is declared as an integer (`int`) but it is tacitly assumed to be non-negative. The signature `char* argv[]` is an archaic way to say that `argv` is an array of unspecified length of character strings. Here it is again tacitly assumed that the length of an array is the value of `argc`. It is precisely the failure of such tacit assumptions which leads to myriad of crashes and security breaches. Running the example above with `argc = 1` while the true length of `argv` is zero would result in either printing out gibberish of the form `Hello, #$%@...` of humongous length or in a segmentation fault (system crash).

Imagine, we could write those “tacit” assumptions explicitly:
```cpp
main(nat argc, string[argc] argv) {
  ...
}
```

Here we assume our language to have types `nat` for non-negative integers, `string` for character strings, `string[n]` for fixed-length arrays of strings, and most importantly we assume we can use the first argument in the signature of the second one: we allow the type `string[argv]` of the second argument to be dependent on the value of the first argument `argc`. Types of fixed-length arrays of any given element type and length are nothing new, C readily supports them: we easily write `int arr[3]` for an array of three integers in C. Yet in C the value inside square brackets has to be a constant. What makes a difference is that we allow value of previous argument to be used in the signature of the next one.

<dl><dt>Definition</dt>
  <dd>A programming language is said to support dependent typing if it allows one or several arguments of a function to be used to specify the types of the following arguments or the return type.</dd>
</dl>

In dependent languages, types are usually written not at the beginning of a declaration (like in `int n`), but at its end at least for functions. For example, in Typescript, Scala, Kotlin, F#, Agda, etc. a declaration of a function returning an integer looks as follows: `get_count() : int`.  
There is a good reason for it: return types are also allowed to depend on the arguments.

```c
generate_random_sequence(nat length) : int[length];
```

With dependent typing one can avoid manual casts (coercing values into the “right” type) altogether.

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
