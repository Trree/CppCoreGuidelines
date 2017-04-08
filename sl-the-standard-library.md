# SL: The Standard Library {#sl-the-standard-library}

Using only the bare language, every task is tedious \(in any language\). Using a suitable library any task can be reasonably simple.

The standard library has steadily grown over the years. Its description in the standard is now larger than that of the language features. So, it is likely that this library section of the guidelines will eventually grow in size to equal or exceed all the rest.

« ??? We need another level of rule numbering ??? »

C++ Standard library component summary:

* [SL.con: Containers](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-con)
* [SL.str: String](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-string)
* [SL.io: Iostream](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-io)
* [SL.regex: Regex](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-regex)
* [SL.chrono: Time](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-chrono)
* [SL.C: The C standard library](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-clib)

Standard-library rule summary:

* [SL.1: Use libraries wherever possible](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rsl-lib)
* [SL.2: Prefer the standard library to other libraries](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rsl-sl)
* ???

### SL.1: Use libraries wherever possible {#sl1--use-libraries-wherever-possible}

##### Reason {#reason-384}

Save time. Don’t re-invent the wheel. Don’t replicate the work of others. Benefit from other people’s work when they make improvements. Help other people when you make improvements.

### SL.2: Prefer the standard library to other libraries {#sl2-prefer-the-standard-library-to-other-libraries}

##### Reason {#reason-385}

More people know the standard library. It is more likely to be stable, well-maintained, and widely available than your own code or most other libraries.

## SL.con: Containers {#slcon-containers}

???

Container rule summary:

* [SL.con.1: Prefer using STL`array`or`vector`instead of a C array](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rsl-arrays)
* [SL.con.2: Prefer using STL`vector`by default unless you have a reason to use a different container](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rsl-vector)
* ???

### SL.con.1: Prefer using STL`array`or`vector`instead of a C array {#slcon1-prefer-using-stl-array-or-vector-instead-of-a-c-array}

##### Reason {#reason-386}

C arrays are less safe, and have no advantages over`array`and`vector`. For a fixed-length array, use`std::array`, which does not degenerate to a pointer when passed to a function and does know its size. Also, like a built-in array, a stack-allocated`std::array`keeps its elements on the stack. For a variable-length array, use`std::vector`, which additionally can change its size and handles memory allocation.

##### Example {#example-328}

```
int v[SIZE];                        // BAD

std::array
<
int, SIZE
>
 w;             // ok

```

##### Example {#example-329}

```
int* v = new int[initial_size];     // BAD, owning raw pointer
delete[] v;                         // BAD, manual delete

std::vector
<
int
>
 w(initial_size);   // ok

```

##### Enforcement {#enforcement-353}

* Flag declaration of a C array inside a function or class that also declares an STL container \(to avoid excessive noisy warnings on legacy non-STL code\). To fix: At least change the C array to a
  `std::array`
  .

### SL.con.2: Prefer using STL`vector`by default unless you have a reason to use a different container {#slcon2-prefer-using-stl-vector-by-default-unless-you-have-a-reason-to-use-a-different-container}

##### Reason {#reason-387}

`vector`and`array`are the only standard containers that offer the fastest general-purpose access \(random access, including being vectorization-friendly\), the fastest default access pattern \(begin-to-end or end-to-begin is prefetcher-friendly\), and the lowest space overhead \(contiguous layout has zero per-element overhead, which is cache-friendly\). Usually you need to add and remove elements from the container, so use`vector`by default; if you don’t need to modify the container’s size, use`array`.

Even when other containers seem more suited, such a`map`for O\(log N\) lookup performance or a`list`for efficient insertion in the middle, a`vector`will usually still perform better for containers up to a few KB in size.

##### Note {#note-298}

`string`should not be used as a container of individual characters. A`string`is a textual string; if you want a container of characters, use`vector</*char_type*/>`or`array</*char_type*/>`instead.

##### Exceptions {#exceptions-1}

If you have a good reason to use another container, use that instead. For example:

* If`vector`suits your needs but you don’t need the container to be variable size, use`array`instead.

* If you want a dictionary-style lookup container that guarantees O\(K\) or O\(log N\) lookups, the container will be larger \(more than a few KB\) and you perform frequent inserts so that the overhead of maintaining a sorted`vector`is infeasible, go ahead and use an`unordered_map`or`map`instead.

##### Enforcement {#enforcement-354}

* Flag a
  `vector`
  whose size never changes after construction \(such as because it’s
  `const`
  or because no non-
  `const`
  functions are called on it\). To fix: Use an
  `array`
  instead.

## SL.str: String {#slstr-string}

???

## SL.io: Iostream {#slio-iostream}

???

Iostream rule summary:

* [SL.io.1: Use character-level input only when you have to](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rio-low)
* [SL.io.2: When reading, always consider ill-formed input](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rio-validate)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [SL.io.50: Avoid`endl`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rio-endl)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)

### SL.io.1: Use character-level input only when you have to {#slio1-use-character-level-input-only-when-you-have-to}

???

### SL.io.2: When reading, always consider ill-formed input {#slio2-when-reading-always-consider-ill-formed-input}

???

### SL.io.50: Avoid`endl` {#slio50-avoid-endl}

### Reason {#reason-388}

The`endl`manipulator is mostly equivalent to`'\n'`and`"\n"`; as most commonly used it simply slows down output by doing redundant`flush()`s. This slowdown can be significant compared to`printf`-style output.

##### Example {#example-330}

```
cout 
<
<
 "Hello, World!" 
<
<
 endl;    // two output operations and a flush
cout 
<
<
 "Hello, World!\n";          // one output operation and no flush

```

##### Note {#note-299}

For`cin`/`cout`\(and equivalent\) interaction, there is no reason to flush; that’s done automatically. For writing to a file, there is rarely a need to`flush`.

##### Note {#note-300}

Apart from the \(occasionally important\) issue of performance, the choice between`'\n'`and`endl`is almost completely aesthetic.

## SL.regex: Regex {#slregex-regex}

???

## SL.chrono: Time {#slchrono-time}

???

## SL.C: The C standard library {#slc-the-c-standard-library}

???

C standard library rule summary:

* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)



