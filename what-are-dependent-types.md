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

§ Refinement types and Witness types
------------------------------------

In the above section we used the type `nat` of non-negative integers, i.e. restriction of integers by a predicate `n >= 0`. Such types are called refinement types:

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

Providing a name for every refinement type would bloat the program, so one also needs a lightweight inline variant like this:
```
f(int i, int j, {i < j})
```

This funcion is supposed to accept an arbitrary inter `i` and an integer `j` greater than `i`. This notation can be also read as
```
f(int i, int j, {i < j} invisible_argument)
```
where `{i < j}` is the type of witnesses that `i < j` returns `true`. It is, `f` is now a function with tree arguments: two arbitrary integers and a witness that the second one is greater than the first one.

<dl><dt>Definition 3</dt>
  <dd><i>Witness type</i> for a given proposition is a type inhabited by certificates that the proposition holds.</dd>
</dl>

In particular, witness types `{expr}` are inhabited by certificates that `expr` evaluates to `true`. In presense of witness types, refinement types can be considered a special case of dependent types.

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
Here `printf_args(template)` is a “type-valued function” that extracts the list of expected types for the additional arguments from the `template`. In our example
```cpp
printf_args("Hello, %s! Current CPU temperature is %f.")
```
would return `(string, float)`.

Here is a typical case of incorrect `printf()` usage, that leads to a security vulnerability:
```cpp
main(int argc, char* argv[]) {
  if (argc != 1) throw InvalidArgumentException;
  printf("Hello " + argv[0] + "!");
}
// WRONG
```
This example terminates with an `InvalidArgumentException` unless called with exactly one command-line parameter. Otherwise it is meant to print `Hello, {first command-line parameter}!`. However, if executed with command-line parameter like `"Robert %d Jones"` it would either crash or read out specitic memory bytes where `printf` would expect its nonexistent addtional argument (due to `%d` in the template) to be stored. With dependently-typed `printf()` this example wouldn't compile because the number of additional `printf()`-arguments and their types cannot be determined in compile-time. In order to make it compile, one has to ensure there are zero additional arguments. For example, like this
```cpp
main(nat argc, string[argc] argv) {
  if (argc != 1) throw InvalidArgumentException;
  if (printf_args(argv[0]) != ()) throw InvalidArgumentException;
  printf("Hello " + argv[0] + "!");
}
```

Of course, one could simply use the solution that we used all in our very first example:
```cpp
main(nat argc, string[argc] argv) {
  if (argc != 1) throw InvalidArgumentException;
  printf("Hello %s!", argv[0]);
}
```

### Eliminating SQL Injection Vulnerability

![https://www.explainxkcd.com/wiki/index.php/Little_Bobby_Tables](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)

For software developers who have experience writing database-facing code, let me mention a very similar use case of profound importance: functions performing requests to relational databases that typically look a lot like `printf()` and have very similar security problems. Let's consider an example:
```kotlin
db.query("SELECT * FROM Records WHERE student = " + student + " year = " + year)
// WRONG!!!

db.query("SELECT * FROM Records WHERE student = ? AND year = ?", student, year)
// ok
```

[<xkcd.com/327>](http://xkcd.com/327/) refers precisely to implementations like the one used in the first line. Running it with student named "Robert'); DROP TABLE Students; --" would instantaneoulsy ruin the whole database. Yet it is possible to eliminate such vulnerabilies by proper typing.

If the database schema is known in advance, one can determine that `query()` has to have two additional arguments of types `string` and `int` by parsing the query. The output type can be determined as well. One can integrate importing of the database schemata into the build process, i.e. fill in the `db.schema` field for the `db` object each time the application is compiled. That way, the following signature for `query()` function can be achieved:

```Kotlin
db.query(string q, <db.query_args(q)> ...args) : <db.query_results(q)> throws IncompatibleDbSchemaException
// The `IncompatibleDbSchema` exception arises if the database schema has
// changed since the application was compiled, so it has to be rebuilt.
```

This way we do not only eliminate security vulnerabilities but also obliviate manual casts and boilerplate classes for object-relational mapping, etc. Results of a query just have the right automatically generated types:
```
foreach (var record in db.query("SELECT * FROM Records WHERE student = ?", student)) {
  printf("Name: %s, Grade average: %f", record.name, record.grade_average);
}
// Here the variable `record` automatically have the type of a record
// with properly typed fields `name`, `grade_average` etc.
```

Precise signatures like these are highly desirable for public APIs and settled libraries. They prevent security vulnerabilities and allow to enforce strict argument validation each time data crosses application boundaries. At the same time they allow to perform validation in compile-time only (when applicable) without any performance penalties for validation. Additionally, they allow API users to perform argument validation beforehand to ensure no InvalidArgumentExceptions could arise.

API discriptions do not include source code of the functions provided by the API, but only the signatures of those functions. However, sources of the type-level functions mentioned in those signatures are a part of the signature, because they have to be executable in compile-time and possibly on a remote machine. Therefore, they have to return a result for all inputs while employing no side effects (no input/output, no exception throwing etc). In “sufficiently powerful” languages the converse is also true: all manifestly terminating side-effect free functions can be cast to type level.

In such languages, signatures can express any restrictions on arguments, however complicated they might be. {TODO: Написать, что и возвращающий}



{**TODO Завершающий абзац:** Закруглить, что последний пример показывает какое у завтипов офигенное прямое практическое приминение, и сказать что their scope goes far beyond this. И вернуться к уровню статьи, что “мы строили-строили и наконец построили”.}
