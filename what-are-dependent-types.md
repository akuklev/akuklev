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
It prints out `"Hello, world!"` if executed without command-line parameters and `"Hello, {first command-line parameter}!"` otherwise.

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

This signature is meant to state that `argc` is a natural number (= non-negative integer), and `argv` a fixed-length array of strings with its length given by `argc`. The real C used to support fixed-length arrays, like `int arr[3]` for an integer array of length 3, but the expression in square brackets had to be a compile-time constant. However, in order to state what we actually know about the length of `argv` we have to allow expressions, values of which are not determined in compile-time. In other words, we have to allow types to depend on "run-time" values — that's where the name “dependent types” comes from. Some programming languages, unlike C, support such signatures, or, more formally:

<dl><dt>Definition 1</dt>
  <dd>A programming language is said be <i>dependently typed</i> if it allows one or several arguments of a function to be used to specify the types of the following arguments or the return type.</dd>
</dl>

Above we handled only the case where an argument of a function (`argc`) is used to specify the type of the following argument (`argv`), while the Def. 1 also mentions return types, so let us provide an example for this case too. In dependently typed languages, return types of functions have to be written not at the beginning of a declaration, but at its end. For example, a declaration of a function returning an integer looks as follows: `get_count() : int`. That's precisely because the return type of a function can also depend on the arguments:
```c
generate_random_sequence(nat length) : int[length];
```

In the examples above we only used the unchanged values arguments as parameters (in particular array length) of types. For reasonable usage flexibility we also need to transform the values, i.e. write signatures like this one
```c
f(nat length) : int[2 * length + 1]
```

Here we use an expression involving the argument `legnth` as parameter of the return type, namely the expression `2 * length + 1`. Are we allowed to use any expressions? It turns out, for the type system to be sound, there is a restriction: in such expressions we are only allowed to use functions that return a result for all inputs while employing no side effects (no input/output, no exception throwing etc). Thus, a language with reasonable support of dependent types has to have means to distinguish such (manifestly terminating side effect-free) functions. Typically, they (the "true" functions in mathematical sense) would be distinguised from “effectful” functions (also called procedures or routines) on the type level, and there would be a way to cast a “procedure” into a ”function” if the compiler manages to check that it employs no side effect and terminates on every valid input. In most dependent languages programers are required to change their ways to program and sometimes to be painfully verbose to convince the compiler that a given function is indeed a function in the mathematical sense. As it turns out, it does not have to be this way as exemplified by [Microsoft Research F*](https://fstar-lang.org/) (one of the most production-ready verification-oriented languages) which makes use of a combination of SMT solving and advanced termination-checking algorithms.


§ Advanced examples
-------------------

Now let's turn our attention to the function `printf()`. In the example above, it was used to print “Hello, world!” and “Hello, {name}!”:
```c
printf( "Hello, world!" ); 
...
printf( "Hello, %s!", argv[0] );
```

The function “print formatted” `printf(string template, ...)` has a variable number of arguments depending on the first argument `template`. If `template` contains no %-patterns, `printf` has no additional arguments. If it has a single `%s`, as in our example, it has an additional argument of type `string`. The pattern `%d` would require an integer argument, and `%f` a `float`. The number of %-patterns in the template determines how many additional arguments are required.

`printf()` has been used for security attacks so frequently that they got a proper name: Format String Vulnerability. All of them could be prevented by a signature making tacit assumptions explicit:
```c
printf(string template, <printf_args(template)> ...args)
```
Here `printf_args(template)` is a “type-valued” (or “type level”) function that extracts the list of expected types for the additional arguments from the `template`. In our example
```cpp
printf_args("Hello, %s! Current CPU temperature is %f.")
```
would return `(string, float)`.

Here is a typical case of incorrect `printf()` usage, that leads to a security vulnerability:
```cpp
main(int argc, char* argv[]) {
  if (argc == 1) printf("Hello " + argv[0] + "!");
}
// WRONG
```
This example checks if it has been called with exactly one command-line parameter and is meant to print `Hello, {first command-line parameter}!` in this case. However, if executed with command-line parameter like `"Robert %d Jones"` it would either crash or read out specitic memory bytes where `printf` would expect its nonexistent addtional argument (due to `%d` in the template) to be stored. With dependently-typed `printf()` this example wouldn't compile because the number of additional `printf()`-arguments and their types cannot be determined in compile-time. In order to make it compile, one has to ensure there are zero additional arguments. For example, like this
```cpp
main(nat argc, string[argc] argv) {
  if (argc == 1 && printf_args(argv[0]) == ()) printf("Hello " + argv[0] + "!");
}
```

Of course, one could also employ the solution that we used all in our very first example:
```cpp
main(nat argc, string[argc] argv) {
  if (argc == 1) printf("Hello %s!", argv[0]);
}
```

**For the ones having experience with database-facing code, let me mention the use case of profound importance:**
![https://www.explainxkcd.com/wiki/index.php/Little_Bobby_Tables](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)


Requests to databases work very similar `printf()` and are prone to the same security problems. Let's consider an example:
```kotlin
db.query("SELECT * FROM Students WHERE (name = '" + name + "' AND year = " + year + ")")
// WRONG!!!

db.query("SELECT * FROM Students WHERE (name = ? AND year = ?)", name, year)
// ok
```

The comic [<xkcd.com/327>](http://xkcd.com/327/) refers to the vulnerability (called SQL Injection Vulnerability) that arises from the code marked `WRONG` above. Running this code for student named "Robert'); DROP TABLE Students; --" would instantaneoulsy ruin the whole database by executing the query
```sql
SELECT * FROM Students WHERE (name = 'Robert'); DROP TABLE Students; -- AND year = 2020)
```
(It first lists all students named Robert and then erases the whole table by executing `DROP TABLE Students` request.)

As in the case of `printf`, such vulnerabilies can be completely eliminated by dependent typing.  
If the database schema is known in advance, one can determine that `query()` has to have two additional arguments of types `string` and `int` by parsing the query. The output type can be determined as well. One can integrate importing of the database schemata into the build process, i.e. fill in the `db.schema` field for the `db` object each time the application is compiled. That way, the following signature for `query()` function can be achieved:

```Kotlin
db.query(string q, <db.query_args(q)> ...args) : <db.query_results(q)> throws IncompatibleDbSchemaException
// The `IncompatibleDbSchema` exception arises if the database schema has
// changed since the application was compiled, so it has to be rebuilt.
```

It does not only eliminate security vulnerabilities but also obliviates manual casts and boilerplate classes for object-relational mapping, etc. Results of a query just have the right automatically generated types:
```
foreach (var student in db.query("SELECT * FROM Students)) {
  printf("Name: %s, Grade average: %f", student.name, student.grade_average);
}
// Here the variable `student` automatically has the type of a record
// with properly typed fields `name`, `grade_average` etc.
```


§ Bottomline
------------

Precise signatures like the ones given above are highly desirable for public APIs and settled libraries: they provide excellent insight for the API and library users, prevent security vulnerabilities, and enforce strict argument validation when data crosses application boundaries, while eliminating time-consuming run-time validation if it can be carried out in compile-time. Additionally, precise signatures allow external API users to perform argument validation beforehand to ensure no unexpected run-time errors due to invalid arguments could arise.

I hope we managed to provide a very short introduction to dependent types and demonstrate their tremendous practical usefulness. Even most basic libraries and APIs cannot be typed precisely without employing dependent types, while in presence of dependent types precise signatures can be given even most involved cases.

In fact, the scope of dependent types goes even far beyond that: together with quotient types and univalent universes (two concepts to which the “HoTT”-part in our research group name refers) they enable arbitrary-precision exact real arithmetics and the whole world of abstract mathematical constructions.
  
  
  
  
  

--------------


§ Addendum: Refinement types and Witness types
----------------------------------------------

The claim that dependent types are sufficient to enforce argument validation of any desired complexity actually requires (mild) additional assumptions. So let's dive into the details if you're interested.

In the first section we used the type `nat` of non-negative integers, which can be understood as restriction of integers by a predicate `n >= 0`. Such types are called refinement types:

<dl><dt>Definition 2</dt>
  <dd><i>Refinement type</i> is a type restricted by a (semidecidable) predicate which is assumed to hold for any element of the refined type.</dd>
</dl>

Examples:
```
nat := {int n | n >= 0}
SortedList<T> := {List<T> list | isSorted(list)}
```

Inhabitants of `SortedList<T>` are thus precisely such lists `List<T>`, that `isSorted(l)` returns `true`. The word `semidecidable` in the Def. 2 refers to the requirement that the predicate is given by a “checking” algorithm. The algorithm is not required to terminate on every input (if it does, the predicate is called decidable). 

With refinement types one can express restrictions on arguments (as we have already done with `argc` by using `nat` instead of `int` as its type) or postconditions when used for return types:
```
sort(List<T> list) : SortedList<T>
```

Finding a name for every refinement type is a bit tedious and bloats the code, so let's introduce a lightweight inline notation:
```
f(int i, int j, {i < j})
```

This funcion is supposed to accept an arbitrary inter `i` and an integer `j` greater than `i`. Yet this notation can be also read as
```
f(int i, int j, {i < j} invisible_argument)
```
where `{i < j}` is the type of witnesses certifying that `i < j` returns `true`. It is, `f` is now a function with tree arguments: two arbitrary integers and a witness that the second one is greater than the first one.

<dl><dt>Definition 3</dt>
  <dd><i>Virtual witness type</i> for a given proposition is a virtual type assumed to be inhabited by certificates that the proposition holds. By "virtual" we mean that arguments and variables of those types are used only in compile-time (for type-checking, termination checking and instantiation) and are so to say erased in the compilation process. Certificates do not "exist" in the run-time, they do not have any representation in terms of real bits and bytes in memory.</dd>
</dl>

In particular, virtual witness types `{expr}` are inhabited if and only if `expr` evaluates to `true`. In presense of witness types, refinement types can be considered a special case of dependent types. Witness types are either present or can be emulated in all dependently-typed languaged known to the authors.
