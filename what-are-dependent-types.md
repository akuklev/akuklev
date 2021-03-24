What are dependent types?
=========================

I work at [HoTT and Dependent Types Group](https://research.jetbrains.org/groups/group-for-dependent-types-and-hott) at [JetBrains Research](https://research.jetbrains.org/). This article is an attempt to introduce _dependent types_ for an interested reader who knows enough [C](https://en.wikipedia.org/wiki/C_(programming_language)) to understand the following “Hello, world!” piece:

**Example 1**
```c
main(int argc, char* argv[]) {
  if (argc == 0) {
    printf( "Hello, world!" ); 
  } else {
    printf( "Hello, %s!", argv[1] );
  }
}
```
This program prints out `"Hello, world!"` if executed without command-line parameters and `"Hello, {first command-line parameter}!"` otherwise.

Let us examine this example. Each C program has a unique function called `main()`. When a program is executed, it is precisely the `main()` function which is being called. `main()` has two arguments:
* `argc`: 'argument count' is the number of command-line arguments; 
* `argv`: 'argument values' is the array containing them, with the addition of the program filename as the first item.

The argument `argc` is declared as an integer (`int argc`) and tacitly assumed to be non-negative. The argument `argv` is introduced by `char* argv[]` which is an archaic way to declare `argv` to be a string array of unspecified length. The length of `argv` is tacitly assumed to be `argc + 1`.

There is a problem with tacit assumptions: they can be easily violated; mostly by mistake, but sometimes also maliciously. Failure of tacit assumptions is responsible for myriads of crashes and vast majority of security vulnerabilities. The mismatch between the value of `argc` and the actual length of `argv` in the example above would result in either printing out gibberish of the form `Hello, %$Gz#H@...` of humongous length or in a segmentation fault (system crash).

The solution is to make such assumptions explicit. It goes beyond the capabilities of C, so let us use C-like pseudocode:
```cpp
main(nat argc, string[argc + 1] argv) {
  ...
}
```

This signature is meant to state that `argc` is a natural number (= non-negative integer), and `argv` a fixed-length array of strings with its length given by `argc + 1`.

The real C used to support fixed-length arrays if their length is a compile-time constant. For instance, by `int arr[3]` one declares `arr` to be an integer array of length 3. To make such signagures possible, one has to go beyond compile-time constants and allow expressions, values of which are not determined in compile-time. In other words, types should be allowed to depend on "run-time" values — that is where the name “dependent types” comes from. Some programming languages, unlike C, support such signatures, or, more formally:

<dl><dt>Definition</dt>
  <dd>A programming language is said be <i>dependently typed</i> if it allows one or several arguments of a function to be used to specify the types of the following arguments or the return type.</dd>
</dl>

Notice, that the definition also mentions the return type, so let us provide an example for this case. In dependently typed languages, the return type of a function has to be written not at the beginning of a declaration, but at its end. For example, a declaration of a function returning an integer looks as follows: `get_count() : int`, where colon `:` separates declarandum `get_count()` and its type `int`. That is precisely because the return type of a function can also depend on the arguments:
```c
generate_random_sequence(nat length) : int[length];
```

§ Advanced examples
-------------------

Now let us take a closer look at the function `printf()` that was used to print “Hello, world!” and “Hello, {name}!” in the Example 1:
```c
printf( "Hello, world!" ); 
...
printf( "Hello, %s!", argv[0] );
```

The function “print formatted” `printf(string template, ...)` has a variable number of arguments depending on the first argument `template`. If `template` contains no %-patterns, `printf` has no additional arguments. If it has a single `%s`, as in the Example 1, it has an additional argument of type `string`. The pattern `%d` would require an integer argument, and `%f` a `float`. The number of %-patterns in the template determines how many additional arguments are required.

Using dependent typing, one can make these tacit assumptions on the number of additional `printf(template, ...)` arguments and their types explicit:
```c
printf(string template, <printf_args(template)> ...args)
```
Here, `printf_args(template)` is a “type-valued” (or “type level”) function that extracts the list of expected types for the additional arguments from the `template`. For instance,
```cpp
printf_args("Hello, %s! Current CPU temperature is %f.")
```
would return `{string, float}`.

Here is a typical case of incorrect `printf()` usage, that leads to a problem:

**Example 2**
```cpp
main(int argc, char* argv[]) {
  if (argc == 1) printf("Hello " + argv[1] + "!");
}
// WRONG
```
This example checks if it has been called with exactly one command-line parameter and is meant to print `Hello, {first command-line parameter}!` in this case. However, if executed with command-line parameter like `"Bobby %d Tables"`, the `printf()` function would expect an additional integer argument but would not find it. Depending on the compiler and environment, it would lead one of several possible outcomes:
* the system would crash,
* the program would crash, or
* the program would print out values of specitic memory bytes where `printf` would expect its nonexistent addtional argument to be stored.

Now let us see how the dependent signature `printf(string template, <printf_args(template)> ...args)` would prevent such behaviour. When the `template` is known in compile time, the compiler would check that `printf()` is applied to the correct number of arguments. However, in `printf("Hello " + argv[0] + "!")` the `template` is not known in compile time, thus, the compiler cannot determine how many additional `printf()` arguments are required, and the example would not compile.

To make it compile, one has to ensure there are zero additional arguments. For example, like this
```cpp
main(nat argc, string[argc + 1] argv) {
  if (argc == 1) {
    string s = "Hello " + argv[1] + "!";
    if (printf_args(s) != {}) {
      printf( "Please don't use any percent characters!" );
    } else {
      printf( s );
    }
  }
}
```

However, the standard solution is the one used in the Example 1:
```cpp
main(nat argc, string[argc] argv) {
  if (argc == 1) printf("Hello %s!", argv[1]);
}
```
This solution has no problems with percent characters. It would just print out `Hello, Bobby %d Tables!`.

**For the ones having experience with database-facing code, let us mention the use case of profound importance:**

<div align="center"><a href="https://www.explainxkcd.com/wiki/index.php/Little_Bobby_Tables"><img src="https://imgs.xkcd.com/comics/exploits_of_a_mom.png" alt="http://xkcd.com/327/ — Little Bobby Tables"></a></div>

Requests to databases work very similar `printf()` and are prone to the same security problems. Let us consider an example:

**Example 3**
```kotlin
db.query("SELECT * FROM Students WHERE (name = '" + name + "' AND year = '" + year + "')")
// WRONG!!!

db.query("SELECT * FROM Students WHERE (name = ? AND year = ?)", name, year)
// ok
```

The comic [<xkcd.com/327>](http://xkcd.com/327/) refers to the vulnerability (called SQL Injection Vulnerability) that arises from the code marked `WRONG` above. Running this code for student named "Robert'); DROP TABLE Students; --" would instantaneoulsy ruin the whole database by executing the query
```sql
SELECT * FROM Students WHERE (name = 'Robert'); DROP TABLE Students; -- AND year = 2020)
```
(It first lists all students named Robert and then erases the whole table by executing `DROP TABLE Students` request.)

As in the case of `printf`, such vulnerabilies can be completely eliminated using dependent typing.  
If the database schema is known in advance, one can determine that `query()` has to have two additional arguments of types `string` and `int` by parsing the query. The output type can be determined as well. One can integrate importing of the database schemata into the build process, i.e. fill in the `db.schema` field for the `db` object each time the application is compiled. That way, the following signature for `query()` function can be achieved:

```Kotlin
db.query(string q, <db.query_args(q)> ...args) : <db.query_results(q)> throws IncompatibleDbSchemaException
// The `IncompatibleDbSchema` exception arises if the database schema has
// changed since the application was compiled, so it has to be rebuilt.
```

Not only does it eliminate security vulnerabilities but also obliviates manual casts and boilerplate classes for object-relational mapping, etc. Results of a query just have the right automatically generated types:

**Example 4**
```
foreach (var student in db.query("SELECT * FROM Students)) {
  printf("Name: %s, Grade average: %f", student.name, student.grade_average);
}
// Here the variable `student` automatically has the type of a record
// with properly typed fields `name`, `grade_average` etc.
```

§ Dependent types and Argument Validation
-----------------------------------------

Besides restricting the types of the arguments, it is often required to restrict their values. Without dependent types, this is typically done as follows:
```
f(int n, int m) {
  assert(n > m);
  ...
}
```
Here, the function `f` checks its arguments satisfy aditional assumption `n > m` on each call and raises an exception if they do not. This prevents security vulnerabilites and facilitates early error detection, but this approach still has several downsides:
1) Time consuming run-time validation is performed even when it is obviously unnecessary. For example, when `f(n, 0)` is called in a branch where `n > 0`. Such cases could be detected and resolved in compile-time.
2) It might be desirable to perform validation before calling the function. Yet the only way to do so involves inspecting the source code of the function being called. In case of foreign libraries or APIs the source code might be unavailable or unexpectedly changed by a third party.

In this approach, the restrictions on the arguments are not reflected on the level of signatures, which is the root cause of both problems.
Thus, they can both be resolved by shifting the restrictions into the signatures. In a C-like pseudocode, it could be expressed as follows:
```
f(int n, int m, n > m) {
  ...
}
```

By making the restrictions a part of the signature, one makes them explicitly visible to the users and to the compiler so that it can optimize away all redundant or unneccessary validations. Not all dependently typed languages support restrictions in function signatures. It is a minor¹ but essential extension.

To keep the signatures reasonably short, one can introduce shorthand names for types with restrictions on values:
```cpp
nat := int n, n >= 0

string<Grammar g> := string s, g.matches(s)

SortedList<Ordered T> := List<T> list, T.isSorted(list)
```

With `string<Grammar g>` from the example above, one can make the signature of former SQL query method even better:
```Kotlin
db.query(string<query> q, <db.query_args(q)> ...args) : <db.query_results(q)> throws IncompatibleDbSchemaException
```

Now that the programming tools (IDEs) can see that the string literal used for the argument `q` should comply with a given grammar, they can also provide the respective inline validation, syntax highlighting, context help, autocompletion, etc.

---
1. From theoretical perspective, this extension does not introduce any additional complexity to the language if it readily supports dependent types and [inductive data types](https://en.wikipedia.org/wiki/Inductive_type).



§ What Makes Dependent Typing Difficult?
----------------------------------------

Having seen how useful dependent typing is, one might wonder why they are not widely adopted. To answer this question, let us take a closer look at the signature
```cpp
main(nat argc, string[argc + 1] argv) {
  ...
}
```

The expression `argc + 1` is used as a parameter for the second argument's type. An expression in such position must be guaranteed to deterministically return a result for all possible values of variables it depends on (all possible values of `argc` in this case).
It is also not allowed to employ any side effects, i.e. it is not allowed to modify any external data, employ any input/output, throw any exceptions etc. Henceforce, such effect-free manifestly terminating functions will be called “pure functions”.

The language could be restricted to pure functions only, but this is hardly an option a general purpose programming language. The other option is to introduce some means for distinguishing pure functions among others. In particular, the language has to have a special type for pure functions and some in-built machinery to check if a given function qualifies as pure, namely:
* a termination checker;
* a side-effect tracking policy;
* optionally an SMT solver which is able to determine that side effects which might happen, say an `IndexOutOfBoundsException` or a `DivisionByZeroException`, actually never happen or at least never leak out;
* a fallback mechanism for cases when termination checker or SMT solver fail to determine the purity of a function.

Fallback mechanisms are unavoidable because of inherent limitations of other mechanisms. Termination checking is known to be undecidable in general, thus, in some non-trivial cases, the termination checker cannot ensure termination automatically (see [Halting Problem](https://en.wikipedia.org/wiki/Halting_problem)). SMT solvers also have their limitations: sometimes they might fail to see why a `DivisionByZeroException` could never arise, even if it is rather obvious to the human programmer. The minimalistic fallback mechanism is to allow the programmers to use special “Trust Me”-directive in such cases, perhaps with a mandatory name of the responsible programmer and a commentary why they assume their code never to perform a division-by-zero in a specific position, and their loops or recursion to terminate.

For critical software one would want something more reliable than "Trust Me"-directives. For this reason, most dependently-typed languages also support certified programming, which presumes a sublanguage for machine-readable proofs. Unfortunatelly, certified programming requires programmers to acquire a new cognitively demanding skill.

The complexity of these mechanisms explains why dependent typing is still not widely adopted in general purpose languages.

§ Concluding Notes
------------------

Dependent types allow to put all the assumptions on arguments into function signatures. This is beneficial for multiple reasons:
* It helps to prevent security vulnerabilies.
* It provides the framework for argument validation that extends the possibilites (**TODO**).
* It shifts critical information from the documentation, which tends to be neglected by both writers and intended readers, into function signatures which do not grow outdated.

Precise signatures made possible by dependent types are highly desirable for APIs and settled libraries, but the scope of dependent types goes far beyond that: they enable a multitude of very advanced programming techniques including exact real arithmetics.  

Dependent typing comes at a cost: design of a dependently typed language and its compiler is a challenging endeavor. Dependent typing also requires the notion of pure functions and is connected to certified programming, but it does not enforce certified or purely functional programming.

We hope, we managed to provide a short introduction to dependent types and demonstrate their tremendous usefulness.
