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

Let us work through this example line-by-line. Each C program has a unique function called `main()`. When a program is executed, it is precisely the `main()` function which is being called. `main()` has two arguments:
* `argc`: 'argument count' is the number of command-line arguments; 
* `argv`: 'argument values' is the array containing them, supplemented by the program filename as zeroth item.

The argument `argc` is declared as an integer (`int argc`) and tacitly assumed to be non-negative. The argument `argv` is introduced by `char* argv[]` which is an <abbr title="C is notorious for allowing typal information on both sidies of declarandum, yielding monsters like 'char *(*(**foo[][8])())[]'">archaic way</a> to declare `argv` to be a string array of unspecified length. The length of `argv` is tacitly assumed to be `argc + 1`.

There is a problem with tacit assumptions: they can be easily violated; mostly by mistake, but sometimes also maliciously. Failure of tacit assumptions is responsible for myriads of crashes and vast majority of security vulnerabilities. Mismatch between the value of `argc` and the actual length of `argv` in the above example would result in either printing out gibberish of the form `Hello, %$Gz#H@...` of humongous length or in a segmentation fault (system crash).

The solution is to make tacit assumptions explicit. It goes beyond the capabilities of C, so let us use C-like pseudocode:
```cpp
main(nat argc, string[argc + 1] argv) {
  ...
}
```

This signature is meant to state that `argc` is a natural number (= non-negative integer), and `argv` a fixed-length array of strings with its length given by `argc + 1`.

The real C used to support fixed-length arrays if their length is a compile-time constant. For instance, by `int arr[3]` one declares `arr` to be an integer array of length 3. To make tacit assumptions explicit, one has to go beyond compile-time constants and allow expressions, values of which are not determined in compile-time. In other words, types should be allowed to depend on "run-time" values — that is where the name “dependent types” comes from. Some programming languages, unlike C, support such signatures, or, more formally:

<dl><dt>Definition 1</dt>
  <dd>A programming language is said be <i>dependently typed</i> if it allows one or several arguments of a function to be used to specify the types of the following arguments or the return type.</dd>
</dl>

The example of dependently-typed signature of `main()` only illustrtes the case where an argument of a function (`argc`) is used to specify the type of the following argument (`argv`). But the definition 1 also mentions the return type, so let us provide an example for this case as well. In dependently typed languages, the return type of a function has to be written not at the beginning of a declaration, but at its end. For example, a declaration of a function returning an integer looks as follows: `get_count() : int`, where colon (:) separates declarandum `get_count()` and its type `int`. That is precisely because the return type of a function can also depend on the arguments:
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

`printf()` has been used for security attacks so frequently that they got a proper name: Format String Vulnerability. All of them could be prevented by a signature making tacit assumptions explicit:
```c
printf(string template, <printf_args(template)> ...args)
```
Here `printf_args(template)` is a “type-valued” (or “type level”) function that extracts the list of expected types for the additional arguments from the `template`. For instance,
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
This example checks if it has been called with exactly one command-line parameter and is meant to print `Hello, {first command-line parameter}!` in this case. However, if executed with command-line parameter like `"Bobby %d Tables"` it would either crash or read out specitic memory bytes where `printf` would expect its nonexistent addtional argument (due to `%d` in the template) to be stored. With dependently-typed `printf()` this example would not compile because the number of additional `printf()`-arguments and their types cannot be determined in compile-time. In order to make it compile, one has to ensure there are zero additional arguments. For example, like this
```cpp
main(nat argc, string[argc + 1] argv) {
  if (argc == 1 && printf_args(argv[1]) == ()) {
    printf("Hello " + argv[0] + "!");
  }
}
```

Of course, one could also employ the solution that used in the Example 1:
```cpp
main(nat argc, string[argc] argv) {
  if (argc == 1) printf("Hello %s!", argv[0]);
}
```

**For the ones having experience with database-facing code, let us mention the use case of profound importance:**

<div align="center"><a href="https://www.explainxkcd.com/wiki/index.php/Little_Bobby_Tables"><img src="https://imgs.xkcd.com/comics/exploits_of_a_mom.png" alt="http://xkcd.com/327/ — Little Bobby Tables"></a></div>

Requests to databases work very similar `printf()` and are prone to the same security problems. Let us consider an example:
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

I hope this article managed to provide a short introduction to dependent types and demonstrate their tremendous practical usefulness. Even most basic libraries and APIs cannot be typed precisely without employing dependent types, while in presence of dependent types precise signatures can be given even most involved cases.   The scope of dependent types goes even far beyond that: they enable a multitude of very advanced programming techniques including exact real arithmetics.  
  
  
<div align="center">* * * * *</div>
  

§ Addendum: What Makes Dependent Typing Complicated?
----------------------------------------------------

Let's take a closer look onto the signature
```cpp
main(nat argc, string[argc + 1] argv) {
  ...
}
```

The expression `argc + 1` is used as a parameter for the second argument's type. An expression in such position is must be guaranteed to deterministically return a result for all possible values of variables it depends on (all possible values of `argc` in this case) while employing no side effects, i.e. without modifying anything outside, without any input/output, without throwing any exceptions etc. 

Thus, a language with reasonable support of dependent types has to have the means to distinguish such expressions. In particular, the language has to have a special type for effect-free manifestly terminating functions (henceforce called “pure functions”), usually denoted `A -> B`. And then the language is either restricted to pure functions only, which is hardly an option a general purpose programming language, or has to have some inbuilt machinery to check if a given function qualifies as pure: a termination checker, a side-effect tracking policy, and optionally an SMT solver which is able to determine that side effects that might happen (say an `IndexOutOfBoundsException` or a `DivisionByZeroException`) actually never happen or at least never leak out to the outer world.

The complexity of such machinery explains why dependent typing are still not widely adopted in general purpose languages.

Furthermoer, termination checking is known to be undecidable in general, thus in some non-trivial cases, the termination checker ensure termination automatically. SMT solvers are not almighty as well: sometimes they might fail to see why a `DivisionByZeroException` could never arise, even if it is rather obvious to the human programmer. One way to deal with such cases is to require the programmer to write a “TrustMe” pragma with their name on them, and a commentary why they assume their code never to perform a division-by-zero in a specific position, and their loops or recursion to terminate. But for critical applications we actually need a whole sublanguage for machine-readable proofs. It does not only make the language more complicated, but also requires programmers to acquire a new cognitively demanding skill.


§ Addendum II: Refinement types and Witness types
-------------------------------------------------

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
