# CPL: C-style programming {#cpl-c-style-programming}

C and C++ are closely related languages. They both originate in “Classic C” from 1978 and have evolved in ISO committees since then. Many attempts have been made to keep them compatible, but neither is a subset of the other.

C rule summary:

* [CPL.1: Prefer C++ to C](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rcpl-C)
* [CPL.2: If you must use C, use the common subset of C and C++, and compile the C code as C++](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rcpl-subset)
* [CPL.3: If you must use C for interfaces, use C++ in the code using such interfaces](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rcpl-interface)

### CPL.1: Prefer C++ to C {#cpl1-prefer-c-to-c}

##### Reason {#reason-369}

C++ provides better type checking and more notational support. It provides better support for high-level programming and often generates faster code.

##### Example {#example-312}

```
char ch = 7;
void* pv = 
&
ch;
int* pi = pv;   // not C++
*pi = 999;      // overwrite sizeof(int) bytes near 
&
ch

```

The rules for implicit casting to and from`void*`in C are subtle and unenforced. In particular, this example violates a rule against converting to a type with stricter alignment.

##### Enforcement {#enforcement-338}

Use a C++ compiler.

### CPL.2: If you must use C, use the common subset of C and C++, and compile the C code as C++ {#cpl2-if-you-must-use-c-use-the-common-subset-of-c-and-c-and-compile-the-c-code-as-c}

##### Reason {#reason-370}

That subset can be compiled with both C and C++ compilers, and when compiled as C++ is better type checked than “pure C.”

##### Example {#example-313}

```
int* p1 = malloc(10 * sizeof(int));                      // not C++
int* p2 = static_cast
<
int*
>
(malloc(10 * sizeof(int)));   // not C, C-style C++
int* p3 = new int[10];                                   // not C
int* p4 = (int*) malloc(10 * sizeof(int));               // both C and C++

```

##### Enforcement {#enforcement-339}

* Flag if using a build mode that compiles code as C.

  * The C++ compiler will enforce that the code is valid C++ unless you use C extension options.

### CPL.3: If you must use C for interfaces, use C++ in the calling code using such interfaces {#cpl3-if-you-must-use-c-for-interfaces-use-c-in-the-calling-code-using-such-interfaces}

##### Reason {#reason-371}

C++ is more expressive than C and offers better support for many types of programming.

##### Example {#example-314}

For example, to use a 3rd party C library or C systems interface, define the low-level interface in the common subset of C and C++ for better type checking. Whenever possible encapsulate the low-level interface in an interface that follows the C++ guidelines \(for better abstraction, memory safety, and resource safety\) and use that C++ interface in C++ code.

##### Example {#example-315}

You can call C from C++:

```
// in C:
double sqrt(double);

// in C++:
extern "C" double sqrt(double);

sqrt(2);

```

##### Example {#example-316}

You can call C++ from C:

```
// in C:
X call_f(struct Y*, int);

// in C++:
extern "C" X call_f(Y* p, int i)
{
    return p-
>
f(i);   // possibly a virtual function call
}

```

##### Enforcement {#enforcement-340}

None needed

# SF: Source files {#sf-source-files}

Distinguish between declarations \(used as interfaces\) and definitions \(used as implementations\). Use header files to represent interfaces and to emphasize logical structure.

Source file rule summary:

* [SF.1: Use a`.cpp`suffix for code files and`.h`for interface files if your project doesn’t already follow another convention](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-file-suffix)
* [SF.2: A`.h`file may not contain object definitions or non-inline function definitions](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-inline)
* [SF.3: Use`.h`files for all declarations used in multiple source files](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-declaration-header)
* [SF.4: Include`.h`files before other declarations in a file](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-include-order)
* [SF.5: A`.cpp`file must include the`.h`file\(s\) that defines its interface](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-consistency)
* [SF.6: Use`using namespace`directives for transition, for foundation libraries \(such as`std`\), or within a local scope \(only\)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using)
* [SF.7: Don’t write`using namespace`in a header file](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using-directive)
* [SF.8: Use`#include`guards for all`.h`files](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-guards)
* [SF.9: Avoid cyclic dependencies among source files](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-cycles)

* [SF.20: Use`namespace`s to express logical structure](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-namespace)
* [SF.21: Don’t use an unnamed \(anonymous\) namespace in a header](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-unnamed)
* [SF.22: Use an unnamed \(anonymous\) namespace for all internal/nonexported entities](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-unnamed2)

### SF.1: Use a`.cpp`suffix for code files and`.h`for interface files if your project doesn’t already follow another convention {#sf1-use-a-cpp-suffix-for-code-files-and-h-for-interface-files-if-your-project-doesnt-already-follow-another-convention}

##### Reason {#reason-372}

It’s a longstanding convention. But consistency is more important, so if your project uses something else, follow that.

##### Note {#note-292}

This convention reflects a common use pattern: Headers are more often shared with C to compile as both C++ and C, which typically uses`.h`, and it’s easier to name all headers`.h`instead of having different extensions for just those headers that are intended to be shared with C. On the other hand, implementation files are rarely shared with C and so should typically be distinguished from`.c`files, so it’s normally best to name all C++ implementation files something else \(such as`.cpp`\).

The specific names`.h`and`.cpp`are not required \(just recommended as a default\) and other names are in widespread use. Examples are`.hh`,`.C`, and`.cxx`. Use such names equivalently. In this document, we refer to`.h`and`.cpp`as a shorthand for header and implementation files, even though the actual extension may be different.

Your IDE \(if you use one\) may have strong opinions about suffices.

##### Example {#example-317}

```
// foo.h:
extern int a;   // a declaration
extern void foo();

// foo.cpp:
int a;   // a definition
void foo() { ++a; }

```

`foo.h`provides the interface to`foo.cpp`. Global variables are best avoided.

##### Example, bad {#example-bad-121}

```
// foo.h:
int a;   // a definition
void foo() { ++a; }

```

`#include <foo.h>`twice in a program and you get a linker error for two one-definition-rule violations.

##### Enforcement {#enforcement-341}

* Flag non-conventional file names.
* Check that
  `.h`
  and
  `.cpp`
  \(and equivalents\) follow the rules below.

### SF.2: A`.h`file may not contain object definitions or non-inline function definitions {#sf2-a-h-file-may-not-contain-object-definitions-or-non-inline-function-definitions}

##### Reason {#reason-373}

Including entities subject to the one-definition rule leads to linkage errors.

##### Example {#example-318}

```
// file.h:
namespace Foo {
    int x = 7;
    int xx() { return x+x; }
}

// file1.cpp:
#include 
<
file.h
>

// ... more ...

 // file2.cpp:
#include 
<
file.h
>

// ... more ...

```

Linking`file1.cpp`and`file2.cpp`will give two linker errors.

**Alternative formulation**: A`.h`file must contain only:

* `#include`
  s of other
  `.h`
  files \(possibly with include guards\)
* templates
* class definitions
* function declarations
* `extern`
  declarations
* `inline`
  function definitions
* `constexpr`
  definitions
* `const`
  definitions
* `using`
  alias definitions
* ???

##### Enforcement {#enforcement-342}

Check the positive list above.

### SF.3: Use`.h`files for all declarations used in multiple source files {#sf3-use-h-files-for-all-declarations-used-in-multiple-source-files}

##### Reason {#reason-374}

Maintainability. Readability.

##### Example, bad {#example-bad-122}

```
// bar.cpp:
void bar() { cout 
<
<
 "bar\n"; }

// foo.cpp:
extern void bar();
void foo() { bar(); }

```

A maintainer of`bar`cannot find all declarations of`bar`if its type needs changing. The user of`bar`cannot know if the interface used is complete and correct. At best, error messages come \(late\) from the linker.

##### Enforcement {#enforcement-343}

* Flag declarations of entities in other source files not placed in a
  `.h`
  .

### SF.4: Include`.h`files before other declarations in a file {#sf4-include-h-files-before-other-declarations-in-a-file}

##### Reason {#reason-375}

Minimize context dependencies and increase readability.

##### Example {#example-319}

```
#include 
<
vector
>

#include 
<
algorithm
>

#include 
<
string
>


// ... my code here ...

```

##### Example, bad {#example-bad-123}

```
#include 
<
vector
>


// ... my code here ...

#include 
<
algorithm
>

#include 
<
string
>
```

##### Note {#note-293}

This applies to both`.h`and`.cpp`files.

##### Note {#note-294}

There is an argument for insulating code from declarations and macros in header files by`#including`headers_after_the code we want to protect \(as in the example labeled “bad”\). However

* that only works for one file \(at one level\): Use that technique in a header included with other headers and the vulnerability reappears.
* a namespace \(an “implementation namespace”\) can protect against many context dependencies.
* full protection and flexibility require
  [modules](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4592.pdf)
  .
  [See also](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0141r0.pdf)
  .

##### Enforcement {#enforcement-344}

Easy.

### SF.5: A`.cpp`file must include the`.h`file\(s\) that defines its interface {#sf5-a-cpp-file-must-include-the-h-files-that-defines-its-interface}

##### Reason {#reason-376}

This enables the compiler to do an early consistency check.

##### Example, bad {#example-bad-124}

```
// foo.h:
void foo(int);
int bar(long);
int foobar(int);

// foo.cpp:
void foo(int) { /* ... */ }
int bar(double) { /* ... */ }
double foobar(int);

```

The errors will not be caught until link time for a program calling`bar`or`foobar`.

##### Example {#example-320}

```
// foo.h:
void foo(int);
int bar(long);
int foobar(int);

// foo.cpp:
#include 
<
foo.h
>


void foo(int) { /* ... */ }
int bar(double) { /* ... */ }
double foobar(int);   // error: wrong return type

```

The return-type error for`foobar`is now caught immediately when`foo.cpp`is compiled. The argument-type error for`bar`cannot be caught until link time because of the possibility of overloading, but systematic use of`.h`files increases the likelihood that it is caught earlier by the programmer.

##### Enforcement {#enforcement-345}

???

### SF.6: Use`using namespace`directives for transition, for foundation libraries \(such as`std`\), or within a local scope \(only\) {#sf6-use-using-namespace-directives-for-transition-for-foundation-libraries-such-as-std-or-within-a-local-scope-only}

##### Reason {#reason-377}

`using namespace`can lead to name clashes, so it should be used sparingly. However, it is not always possible to qualify every name from a namespace in user code \(e.g., during transition\) and sometimes a namespace is so fundamental and prevalent in a code base, that consistent qualification would be verbose and distracting.

##### Example {#example-321}

```
#include
<
string
>

#include
<
vector
>

#include
<
iostream
>

#include
<
memory
>

#include
<
algorithm
>


using namespace std;

// ...

```

Here \(obviously\), the standard library is used pervasively and apparantly no other library is used, so requiring`std::`everywhere could be distracting.

##### Example {#example-322}

The use of`using namespace std;`leaves the programmer open to a name clash with a name from the standard library

```
#include
<
cmath
>

using namespace std;

int g(int x)
{
    int sqrt = 7;
    // ...
    return sqrt(x); // error
}

```

However, this is not particularly likely to lead to a resolution that is not an error and people who use`using namespace std`are supposed to know about`std`and about this risk.

##### Note {#note-295}

A`.cpp`file is a form of local scope. There is little difference in the opportunities for name clashes in an N-line`.cpp`containing a`using namespace X`, an N-line function containing a`using namespace X`, and M functions each containing a`using namespace X`with N lines of code in total.

##### Note {#note-296}

[Don’t write`using namespace`in a header file](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using-directive).

##### Enforcement {#enforcement-346}

Flag multiple`using namespace`directives for different namespaces in a single sourcefile.

### SF.7: Don’t write`using namespace`in a header file {#sf7-dont-write-using-namespace-in-a-header-file}

##### Reason {#reason-378}

Doing so takes away an`#include`r’s ability to effectively disambiguate and to use alternatives.

##### Example {#example-323}

```
// bad.h
#include 
<
iostream
>

using namespace std; // bad

// user.cpp
#include "bad.h"

bool copy(/*... some parameters ...*/);    // some function that happens to be named copy

int main() {
    copy(/*...*/);    // now overloads local ::copy and std::copy, could be ambiguous
}

```

##### Enforcement {#enforcement-347}

Flag`using namespace`at global scope in a header file.

### SF.8: Use`#include`guards for all`.h`files {#sf8-use-include-guards-for-all-h-files}

##### Reason {#reason-379}

To avoid files being`#include`d several times.

##### Example {#example-324}

```
// file foobar.h:
#ifndef FOOBAR_H
#define FOOBAR_H
// ... declarations ...
#endif // FOOBAR_H

```

##### Enforcement {#enforcement-348}

Flag`.h`files without`#include`guards.

### SF.9: Avoid cyclic dependencies among source files {#sf9-avoid-cyclic-dependencies-among-source-files}

##### Reason {#reason-380}

Cycles complicates comprehension and slows down compilation. Complicates conversion to use language-supported modules \(when they become available\).

##### Note {#note-297}

Eliminate cycles; don’t just break them with`#include`guards.

##### Example, bad {#example-bad-125}

```
// file1.h:
#include "file2.h"

// file2.h:
#include "file3.h"

// file3.h:
#include "file1.h"

```

##### Enforcement {#enforcement-349}

Flag all cycles.

### SF.20: Use`namespace`s to express logical structure {#sf20-use-namespaces-to-express-logical-structure}

##### Reason {#reason-381}

???

##### Example {#example-325}

```
???

```

##### Enforcement {#enforcement-350}

???

### SF.21: Don’t use an unnamed \(anonymous\) namespace in a header {#sf21-dont-use-an-unnamed-anonymous-namespace-in-a-header}

##### Reason {#reason-382}

It is almost always a bug to mention an unnamed namespace in a header file.

##### Example {#example-326}

```
???

```

##### Enforcement {#enforcement-351}

* Flag any use of an anonymous namespace in a header file.

### SF.22: Use an unnamed \(anonymous\) namespace for all internal/nonexported entities {#sf22-use-an-unnamed-anonymous-namespace-for-all-internalnonexported-entities}

##### Reason {#reason-383}

Nothing external can depend on an entity in a nested unnamed namespace. Consider putting every definition in an implementation source file in an unnamed namespace unless that is defining an “external/exported” entity.

##### Example {#example-327}

An API class and its members can’t live in an unnamed namespace; but any “helper” class or function that is defined in an implementation source file should be at an unnamed namespace scope.

```
???

```

##### Enforcement {#enforcement-352}

* ???

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

  
A: Architectural Ideas

This section contains ideas about higher-level architectural ideas and libraries.

Architectural rule summary:

* [A.1 Separate stable from less stable part of code](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ra-stable)
* [A.2 Express potentially reusable parts as a library](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ra-lib)
* [A.4 There should be no cycles among libraries](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#?Ra-dag)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)
* [???](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)

### A.1 Separate stable from less stable part of code {#a1-separate-stable-from-less-stable-part-of-code}

???

### A.2 Express potentially reusable parts as a library {#a2-express-potentially-reusable-parts-as-a-library}

##### Reason {#reason-389}

##### Note {#note-301}

A library is a collection of declarations and definitions maintained, documented, and shipped together. A library could be a set of headers \(a “header only library”\) or a set of headers plus a set of object files. A library can be statically or dynamically linked into a program, or it may be`#included`

### A.4 There should be no cycles among libraries {#a4-there-should-be-no-cycles-among-libraries}

##### Reason {#reason-390}

* A cycle implies complication of the build process.
* Cycles are hard to understand and may introduce indeterminism \(unspecified behavior\).

##### Note {#note-302}

A library can contain cyclic references in the definition of its components. For example:

```
???

```

However, a library should not depend on another that depends on it.

# NR: Non-Rules and myths {#nr-non-rules-and-myths}

This section contains rules and guidelines that are popular somewhere, but that we deliberately don’t recommend. We know full well that there have been times and places where these rules made sense, and we have used them ourselves at times. However, in the context of the styles of programming we recommend and support with the guidelines, these “non-rules” would do harm.

Even today, there can be contexts where the rules make sense. For example, lack of suitable tool support can make exceptions unsuitable in hard-real-time systems, but please don’t blindly trust “common wisdom” \(e.g., unsupported statements about “efficiency”\); such “wisdom” may be based on decades-old information or experienced from languages with very different properties than C++ \(e.g., C or Java\).

The positive arguments for alternatives to these non-rules are listed in the rules offered as “Alternatives”.

Non-rule summary:

* [NR.1: Don’t: All declarations should be at the top of a function](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-top)
* [NR.2: Don’t: Have only a single`return`-statement in a function](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-single-return)
* [NR.3: Don’t: Don’t use exceptions](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-no-exceptions)
* [NR.4: Don’t: Place each class declaration in its own source file](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-lots-of-files)
* [NR.5: Don’t: Don’t do substantive work in a constructor; instead use two-phase initialization](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-two-phase-init)
* [NR.6: Don’t: Place all cleanup actions at the end of a function and`goto exit`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-goto-exit)
* [NR.7: Don’t: Make all data members`protected`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-protected-data)
* ???

### NR.1: Don’t: All declarations should be at the top of a function {#nr1-dont-all-declarations-should-be-at-the-top-of-a-function}

##### Reason \(not to follow this rule\) {#reason-not-to-follow-this-rule}

This rule is a legacy of old programming languages that didn’t allow initialization of variables and constants after a statement. This leads to longer programs and more errors caused by uninitialized and wrongly initialized variables.

##### Example, bad {#example-bad-126}

```
???

```

The larger the distance between the uninitialized variable and its use, the larger the chance of a bug. Fortunately, compilers catch many “used before set” errors.

##### Alternative {#alternative-10}

* [Always initialize an object](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-always)
* [ES.21: Don’t introduce a variable \(or constant\) before you need to use it](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-introduce)

### NR.2: Don’t: Have only a single`return`-statement in a function {#nr2-dont-have-only-a-single-return-statement-in-a-function}

##### Reason \(not to follow this rule\) {#reason-not-to-follow-this-rule-1}

The single-return rule can lead to unnecessarily convoluted code and the introduction of extra state variables. In particular, the single-return rule makes it harder to concentrate error checking at the top of a function.

##### Example {#example-331}

```
template
<
class T
>

//  requires Number
<
T
>

string sign(T x)
{
    if (x 
<
 0)
        return "negative";
    else if (x 
>
 0)
        return "positive";
    return "zero";
}

```

to use a single return only we would have to do something like

```
template
<
class T
>

//  requires Number
<
T
>

string sign(T x)        // bad
{
    string res;
    if (x 
<
 0)
        res = "negative";
    else if (x 
>
 0)
        res = "positive";
    else
        res = "zero";
    return res;
}

```

This is both longer and likely to be less efficient. The larger and more complicated the function is, the more painful the workarounds get. Of course many simple functions will naturally have just one`return`because of their simpler inherent logic.

##### Example {#example-332}

```
int index(const char* p)
{
    if (p == nullptr) return -1;  // error indicator: alternatively "throw nullptr_error{}"
    // ... do a lookup to find the index for p
    return i;
}

```

If we applied the rule, we’d get something like

```
int index2(const char* p)
{
    int i;
    if (p == nullptr)
        i = -1;  // error indicator
    else {
        // ... do a lookup to find the index for p
    }
    return i;
}

```

Note that we \(deliberately\) violated the rule against uninitialized variables because this style commonly leads to that. Also, this style is a temptation to use the[goto exit](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rnr-goto-exit)non-rule.

##### Alternative {#alternative-11}

* Keep functions short and simple
* Feel free to use multiple
  `return`
  statements \(and to throw exceptions\).

### NR.3: Don’t: Don’t use exceptions {#nr3-dont-dont-use-exceptions}

##### Reason \(not to follow this rule\) {#reason-not-to-follow-this-rule-2}

There seem to be three main reasons given for this non-rule:

* exceptions are inefficient
* exceptions lead to leaks and errors
* exception performance is not predictable

There is no way we can settle this issue to the satisfaction of everybody. After all, the discussions about exceptions have been going on for 40+ years. Some languages cannot be used without exceptions, but others do not support them. This leads to strong traditions for the use and non-use of exceptions, and to heated debates.

However, we can briefly outline why we consider exceptions the best alternative for general-purpose programming and in the context of these guidelines. Simple arguments for and against are often inconclusive. There are specialized applications where exceptions indeed can be inappropriate \(e.g., hard-real time systems without support for reliable estimates of the cost of handling an exception\).

Consider the major objections to exceptions in turn

* Exceptions are inefficient: Compared to what? When comparing make sure that the same set of errors are handled and that they are handled equivalently. In particular, do not compare a program that immediately terminate on seeing an error with a program that carefully cleans up resources before logging an error. Yes, some systems have poor exception handling implementations; sometimes, such implementations force us to use other error-handling approaches, but that’s not a fundamental problem with exceptions. When using an efficiency argument - in any context - be careful that you have good data that actually provides insight into the problem under discussion.
* Exceptions lead to leaks and errors. They do not. If your program is a rat’s nest of pointers without an overall strategy for resource management, you have a problem whatever you do. If your system consists of a million lines of such code, you probably will not be able to use exceptions, but that’s a problem with excessive and undisciplined pointer use, rather than with exceptions. In our opinion, you need RAII to make exception-based error handling simple and safe – simpler and safer than alternatives.
* Exception performance is not predictable If you are in a hard-real-time system where you must guarantee completion of a task in a given time, you need tools to back up such guarantees. As far as we know such tools are not available \(at least not to most programmers\).

Many, possibly most, problems with exceptions stem from historical needs to interact with messy old code.

The fundamental arguments for the use of exceptions are

* They clearly separates error return from ordinary return
* They cannot be forgotten or ignored
* They can be used systematically

Remember

* Exceptions are for reporting errors \(in C++; other languages can have different uses for exceptions\).
* Exceptions are not for errors that can be handled locally.
* Don’t try to catch every exception in every function \(that’s tedious, clumsy, and leads to slow code\).
* Exceptions are not for errors that require instant termination of a module/system after a non-recoverable error.

##### Example {#example-333}

```
???

```

##### Alternative {#alternative-12}

* [RAII](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-raii)
* Contracts/assertions: Use GSL’s
  `Expects`
  and
  `Ensures`
  \(until we get language support for contracts\)

### NR.4: Don’t: Place each class declaration in its own source file {#nr4-dont-place-each-class-declaration-in-its-own-source-file}

##### Reason \(not to follow this rule\) {#reason-not-to-follow-this-rule-3}

The resulting number of files are hard to manage and can slow down compilation. Individual classes are rarely a good logical unit of maintenance and distribution.

##### Example {#example-334}

```
???

```

##### Alternative {#alternative-13}

* Use namespaces containing logically cohesive sets of classes and functions.

### NR.5: Don’t: Don’t do substantive work in a constructor; instead use two-phase initialization {#nr5-dont-dont-do-substantive-work-in-a-constructor-instead-use-two-phase-initialization}

##### Reason \(not to follow this rule\) {#reason-not-to-follow-this-rule-4}

Following this rule leads to weaker invariants, more complicated code \(having to deal with semi-constructed objects\), and errors \(when we didn’t deal correctly with semi-constructed objects consistently\).

##### Example {#example-335}

```
???

```

##### Alternative {#alternative-14}

* Always establish a class invariant in a constructor.
* Don’t define an object before it is needed.

### NR.6: Don’t: Place all cleanup actions at the end of a function and`goto exit` {#nr6-dont-place-all-cleanup-actions-at-the-end-of-a-function-and-goto-exit}

##### Reason \(not to follow this rule\) {#reason-not-to-follow-this-rule-5}

`goto`is error-prone. This technique is a pre-exception technique for RAII-like resource and error handling.

##### Example, bad {#example-bad-127}

```
void do_something(int n)
{
    if (n 
<
 100) goto exit;
    // ...
    int* p = (int*) malloc(n);
    // ...
    if (some_ error) goto_exit;
    // ...
exit:
    free(p);
}

```

and spot the bug.

##### Alternative {#alternative-15}

* Use exceptions and
  [RAII](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-raii)
* for non-RAII resources, use
  [`finally`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-finally)
  .

### NR.7: Don’t: Make all data members`protected` {#nr7-dont-make-all-data-members-protected}

##### Reason \(not to follow this rule\) {#reason-not-to-follow-this-rule-6}

`protected`data is a source of errors.`protected`data can be manipulated from an unbounded amount of code in various places.`protected`data is the class hierarchy equivalent to global data.

##### Example {#example-336}

```
???

```

##### Alternative {#alternative-16}

* [Make member data`public`or \(preferably\)`private`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-protected)

# RF: References {#rf-references}

Many coding standards, rules, and guidelines have been written for C++, and especially for specialized uses of C++. Many

* focus on lower-level issues, such as the spelling of identifiers
* are written by C++ novices
* see “stopping programmers from doing unusual things” as their primary aim
* aim at portability across many compilers \(some 10 years old\)
* are written to preserve decades old code bases
* aim at a single application domain
* are downright counterproductive
* are ignored \(must be ignored by programmers to get their work done well\)

A bad coding standard is worse than no coding standard. However an appropriate set of guidelines are much better than no standards: “Form is liberating.”

Why can’t we just have a language that allows all we want and disallows all we don’t want \(“a perfect language”\)? Fundamentally, because affordable languages \(and their tool chains\) also serve people with needs that differ from yours and serve more needs than you have today. Also, your needs change over time and a general-purpose language is needed to allow you to adapt. A language that is ideal for today would be overly restrictive tomorrow.

Coding guidelines adapt the use of a language to specific needs. Thus, there cannot be a single coding style for everybody. We expect different organizations to provide additions, typically with more restrictions and firmer style rules.

Reference sections:

* [RF.rules: Coding rules](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-rules)
* [RF.books: Books with coding guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-books)
* [RF.C++: C++ Programming \(C++11/C++14\)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-Cplusplus)
* [RF.web: Websites](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-web)
* [RS.video: Videos about “modern C++”](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-vid)
* [RF.man: Manuals](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-man)

## RF.rules: Coding rules {#rfrules-coding-rules}

* [Boost Library Requirements and Guidelines](http://www.boost.org/development/requirements.html)
  . ???.
* [Bloomberg: BDE C++ Coding](https://github.com/bloomberg/bde/wiki/CodingStandards.pdf)
  . Has a strong emphasis on code organization and layout.
* Facebook: ???
* [GCC Coding Conventions](https://gcc.gnu.org/codingconventions.html)
  . C++03 and \(reasonably\) a bit backwards looking.
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
  . Geared toward C++03 and \(also\) older code bases. Google experts are now actively collaborating here on helping to improve these Guidelines, and hopefully to merge efforts so these can be a modern common set they could also recommend.
* [JSF++: JOINT STRIKE FIGHTER AIR VEHICLE C++ CODING STANDARDS](http://www.stroustrup.com/JSF-AV-rules.pdf)
  . Document Number 2RDU00001 Rev C. December 2005. For flight control software. For hard real time. This means that it is necessarily very restrictive \(“if the program fails somebody dies”\). For example, no free store allocation or deallocation may occur after the plane takes off \(no memory overflow and no fragmentation allowed\). No exception may be used \(because there was no available tool for guaranteeing that an exception would be handled within a fixed short time\). Libraries used have to have been approved for mission critical applications. Any similarities to this set of guidelines are unsurprising because Bjarne Stroustrup was an author of JSF++. Recommended, but note its very specific focus.
* [Mozilla Portability Guide](https://developer.mozilla.org/en-US/docs/Mozilla/C%2B%2B_Portability_Guide)
  . As the name indicates, this aims for portability across many \(old\) compilers. As such, it is restrictive.
* [Geosoft.no: C++ Programming Style Guidelines](http://geosoft.no/development/cppstyle.html)
  . ???.
* [Possibility.com: C++ Coding Standard](http://www.possibility.com/Cpp/CppCodingStandard.html)
  . ???.
* [SEI CERT: Secure C++ Coding Standard](https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=637)
  . A very nicely done set of rules \(with examples and rationales\) done for security-sensitive code. Many of their rules apply generally.
* [High Integrity C++ Coding Standard](http://www.codingstandard.com/)
  .
* [llvm](http://llvm.org/docs/CodingStandards.html)
  . Somewhat brief, pre-C++11, and \(not unreasonably\) adjusted to its domain.
* ???

## RF.books: Books with coding guidelines {#rfbooks-books-with-coding-guidelines}

* [Meyers96](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Meyers96)
  Scott Meyers:
  _More Effective C++_
  . Addison-Wesley 1996.
* [Meyers97](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Meyers97)
  Scott Meyers:
  _Effective C++, Second Edition_
  . Addison-Wesley 1997.
* [Meyers01](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Meyers01)
  Scott Meyers:
  _Effective STL_
  . Addison-Wesley 2001.
* [Meyers05](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Meyers05)
  Scott Meyers:
  _Effective C++, Third Edition_
  . Addison-Wesley 2005.
* [Meyers15](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Meyers15)
  Scott Meyers:
  _Effective Modern C++_
  . O’Reilly 2015.
* [SuttAlex05](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SuttAlex05)
  Sutter and Alexandrescu:
  _C++ Coding Standards_
  . Addison-Wesley 2005. More a set of meta-rules than a set of rules. Pre-C++11.
* [Stroustrup05](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Stroustrup05)
  Bjarne Stroustrup:
  [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf)
  . LCSD05. October 2005.
* [Stroustrup14](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Stroustrup05)
  Stroustrup:
  [A Tour of C++](http://www.stroustrup.com/Tour.html)
  . Addison Wesley 2014. Each chapter ends with an advice section consisting of a set of recommendations.
* [Stroustrup13](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Stroustrup13)
  Stroustrup:
  [The C++ Programming Language \(4th Edition\)](http://www.stroustrup.com/4th.html)
  . Addison Wesley 2013. Each chapter ends with an advice section consisting of a set of recommendations.
* Stroustrup:
  [Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)
  for
  [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html)
  . Mostly low-level naming and layout rules. Primarily a teaching tool.

## RF.C++: C++ Programming \(C++11/C++14\) {#rfc-c-programming-c11c14}

* [TC++PL4](http://www.stroustrup.com/4th.html)
  : A thorough description of the C++ language and standard libraries for experienced programmers.
* [Tour++](http://www.stroustrup.com/Tour.html)
  : An overview of the C++ language and standard libraries for experienced programmers.
* [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html)
  : A textbook for beginners and relative novices.

## RF.web: Websites {#rfweb-websites}

* [isocpp.org](https://isocpp.org/)
* [Bjarne Stroustrup’s home pages](http://www.stroustrup.com/)
* [WG21](http://www.open-std.org/jtc1/sc22/wg21/)
* [Boost](http://www.boost.org/)
* [Adobe open source](http://www.adobe.com/open-source.html)
* [Poco libraries](http://pocoproject.org/)
* Sutter’s Mill?
* ???

## RS.video: Videos about “modern C++” {#rsvideo-videos-about-modern-c}

* Bjarne Stroustrup:
  [C++11 Style](http://channel9.msdn.com/Events/GoingNative/GoingNative-2012/Keynote-Bjarne-Stroustrup-Cpp11-Style)
  . 2012.
* Bjarne Stroustrup:
  [The Essence of C++: With Examples in C++84, C++98, C++11, and C++14](http://channel9.msdn.com/Events/GoingNative/2013/Opening-Keynote-Bjarne-Stroustrup)
  . 2013
* All the talks from
  [CppCon ‘14](https://isocpp.org/blog/2014/11/cppcon-videos-c9)
* Bjarne Stroustrup:
  [The essence of C++](https://www.youtube.com/watch?v=86xWVb4XIyE)
  at the University of Edinburgh. 2014.
* Sutter: ???
* CppCon 15
* ??? C++ Next
* ??? Meting C++
* ??? more ???

## RF.man: Manuals {#rfman-manuals}

* ISO C++ Standard C++11.
* ISO C++ Standard C++14.
* [ISO C++ Standard C++17 CD](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4606.pdf)
  . Committee Draft.
* [Palo Alto “Concepts” TR](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf)
  .
* [ISO C++ Concepts TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf)
  .
* [WG21 Ranges report](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
  . Draft.

## Acknowledgements {#acknowledgements}

Thanks to the many people who contributed rules, suggestions, supporting information, references, etc.:

* Peter Juhl
* Neil MacIntosh
* Axel Naumann
* Andrew Pardoe
* Gabriel Dos Reis
* Zhuang, Jiangang \(Jeff\)
* Sergey Zubkov

and see the contributor list on the github.

  


