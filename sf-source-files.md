
# <span id="S-source"></span>SF: Source files

Distinguish between declarations (used as interfaces) and definitions (used as implementations).
Use header files to represent interfaces and to emphasize logical structure.

Source file rule summary:

* [SF.1: Use a `.cpp` suffix for code files and `.h` for interface files if your project doesn't already follow another convention](#Rs-file-suffix)
* [SF.2: A `.h` file may not contain object definitions or non-inline function definitions](#Rs-inline)
* [SF.3: Use `.h` files for all declarations used in multiple source files](#Rs-declaration-header)
* [SF.4: Include `.h` files before other declarations in a file](#Rs-include-order)
* [SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface](#Rs-consistency)
* [SF.6: Use `using namespace` directives for transition, for foundation libraries (such as `std`), or within a local scope (only)](#Rs-using)
* [SF.7: Don't write `using namespace` in a header file](#Rs-using-directive)
* [SF.8: Use `#include` guards for all `.h` files](#Rs-guards)
* [SF.9: Avoid cyclic dependencies among source files](#Rs-cycles)

* [SF.20: Use `namespace`s to express logical structure](#Rs-namespace)
* [SF.21: Don't use an unnamed (anonymous) namespace in a header](#Rs-unnamed)
* [SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities](#Rs-unnamed2)

### <span id="Rs-file-suffix"></span>SF.1: Use a `.cpp` suffix for code files and `.h` for interface files if your project doesn't already follow another convention

##### Reason

It's a longstanding convention.
But consistency is more important, so if your project uses something else, follow that.

##### Note

This convention reflects a common use pattern:
Headers are more often shared with C to compile as both C++ and C, which typically uses `.h`,
and it's easier to name all headers `.h` instead of having different extensions for just those headers that are intended to be shared with C.
On the other hand, implementation files are rarely shared with C and so should typically be distinguished from `.c` files,
so it's normally best to name all C++ implementation files something else (such as `.cpp`).

The specific names `.h` and `.cpp` are not required (just recommended as a default) and other names are in widespread use.
Examples are `.hh`, `.C`, and `.cxx`. Use such names equivalently.
In this document, we refer to `.h` and `.cpp` as a shorthand for header and implementation files,
even though the actual extension may be different.

Your IDE (if you use one) may have strong opinions about suffices.

##### Example

    // foo.h:
    extern int a;   // a declaration
    extern void foo();

    // foo.cpp:
    int a;   // a definition
    void foo() { ++a; }

`foo.h` provides the interface to `foo.cpp`. Global variables are best avoided.

##### Example, bad

    // foo.h:
    int a;   // a definition
    void foo() { ++a; }

`#include <foo.h>` twice in a program and you get a linker error for two one-definition-rule violations.

##### Enforcement

* Flag non-conventional file names.
* Check that `.h` and `.cpp` (and equivalents) follow the rules below.

### <span id="Rs-inline"></span>SF.2: A `.h` file may not contain object definitions or non-inline function definitions

##### Reason

Including entities subject to the one-definition rule leads to linkage errors.

##### Example

    // file.h:
    namespace Foo {
        int x = 7;
        int xx() { return x+x; }
    }

    // file1.cpp:
    #include <file.h>
    // ... more ...

     // file2.cpp:
    #include <file.h>
    // ... more ...

Linking `file1.cpp` and `file2.cpp` will give two linker errors.

**Alternative formulation**: A `.h` file must contain only:

* `#include`s of other `.h` files (possibly with include guards)
* templates
* class definitions
* function declarations
* `extern` declarations
* `inline` function definitions
* `constexpr` definitions
* `const` definitions
* `using` alias definitions
* ???

##### Enforcement

Check the positive list above.

### <span id="Rs-declaration-header"></span>SF.3: Use `.h` files for all declarations used in multiple source files

##### Reason

Maintainability. Readability.

##### Example, bad

    // bar.cpp:
    void bar() { cout << "bar\n"; }

    // foo.cpp:
    extern void bar();
    void foo() { bar(); }

A maintainer of `bar` cannot find all declarations of `bar` if its type needs changing.
The user of `bar` cannot know if the interface used is complete and correct. At best, error messages come (late) from the linker.

##### Enforcement

* Flag declarations of entities in other source files not placed in a `.h`.

### <span id="Rs-include-order"></span>SF.4: Include `.h` files before other declarations in a file

##### Reason

Minimize context dependencies and increase readability.

##### Example

    #include <vector>
    #include <algorithm>
    #include <string>

    // ... my code here ...

##### Example, bad

    #include <vector>

    // ... my code here ...

    #include <algorithm>
    #include <string>

##### Note

This applies to both `.h` and `.cpp` files.

##### Note

There is an argument for insulating code from declarations and macros in header files by `#including` headers *after* the code we want to protect
(as in the example labeled "bad").
However

* that only works for one file (at one level): Use that technique in a header included with other headers and the vulnerability reappears.
* a namespace (an "implementation namespace") can protect against many context dependencies.
* full protection and flexibility require [modules](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4592.pdf).
[See also](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0141r0.pdf).


##### Enforcement

Easy.

### <span id="Rs-consistency"></span>SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface

##### Reason

This enables the compiler to do an early consistency check.

##### Example, bad

    // foo.h:
    void foo(int);
    int bar(long);
    int foobar(int);

    // foo.cpp:
    void foo(int) { /* ... */ }
    int bar(double) { /* ... */ }
    double foobar(int);

The errors will not be caught until link time for a program calling `bar` or `foobar`.

##### Example

    // foo.h:
    void foo(int);
    int bar(long);
    int foobar(int);

    // foo.cpp:
    #include <foo.h>

    void foo(int) { /* ... */ }
    int bar(double) { /* ... */ }
    double foobar(int);   // error: wrong return type

The return-type error for `foobar` is now caught immediately when `foo.cpp` is compiled.
The argument-type error for `bar` cannot be caught until link time because of the possibility of overloading, but systematic use of `.h` files increases the likelihood that it is caught earlier by the programmer.

##### Enforcement

???

### <span id="Rs-using"></span>SF.6: Use `using namespace` directives for transition, for foundation libraries (such as `std`), or within a local scope (only)

##### Reason

 `using namespace` can lead to name clashes, so it should be used sparingly.
 However, it is not always possible to qualify every name from a namespace in user code (e.g., during transition)
 and sometimes a namespace is so fundamental and prevalent in a code base, that consistent qualification would be verbose and distracting.

##### Example

    #include<string>
    #include<vector>
    #include<iostream>
    #include<memory>
    #include<algorithm>

    using namespace std;

    // ...

Here (obviously), the standard library is used pervasively and apparantly no other library is used, so requiring `std::` everywhere
could be distracting.

##### Example

The use of `using namespace std;` leaves the programmer open to a name clash with a name from the standard library

    #include<cmath>
    using namespace std;

    int g(int x)
    {
        int sqrt = 7;
        // ...
        return sqrt(x); // error
    }

However, this is not particularly likely to lead to a resolution that is not an error and
people who use `using namespace std` are supposed to know about `std` and about this risk.

##### Note

A `.cpp` file is a form of local scope.
There is little difference in the opportunities for name clashes in an N-line `.cpp` containing a `using namespace X`,
an N-line function containing a `using namespace X`,
and M functions each containing a `using namespace X`with N lines of code in total.

##### Note

[Don't write `using namespace` in a header file](#Rs-using-directive).

##### Enforcement

Flag multiple `using namespace` directives for different namespaces in a single sourcefile.

### <span id="Rs-using-directive"></span>SF.7: Don't write `using namespace` in a header file

##### Reason

Doing so takes away an `#include`r's ability to effectively disambiguate and to use alternatives.

##### Example

    // bad.h
    #include <iostream>
    using namespace std; // bad

    // user.cpp
    #include "bad.h"
    
    bool copy(/*... some parameters ...*/);    // some function that happens to be named copy

    int main() {
        copy(/*...*/);    // now overloads local ::copy and std::copy, could be ambiguous
    }

##### Enforcement

Flag `using namespace` at global scope in a header file.

### <span id="Rs-guards"></span>SF.8: Use `#include` guards for all `.h` files

##### Reason

To avoid files being `#include`d several times.

##### Example

    // file foobar.h:
    #ifndef FOOBAR_H
    #define FOOBAR_H
    // ... declarations ...
    #endif // FOOBAR_H

##### Enforcement

Flag `.h` files without `#include` guards.

### <span id="Rs-cycles"></span>SF.9: Avoid cyclic dependencies among source files

##### Reason

Cycles complicates comprehension and slows down compilation.
Complicates conversion to use language-supported modules (when they become available).

##### Note

Eliminate cycles; don't just break them with `#include` guards.

##### Example, bad

    // file1.h:
    #include "file2.h"

    // file2.h:
    #include "file3.h"

    // file3.h:
    #include "file1.h"

##### Enforcement

Flag all cycles.

### <span id="Rs-namespace"></span>SF.20: Use `namespace`s to express logical structure

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <span id="Rs-unnamed"></span>SF.21: Don't use an unnamed (anonymous) namespace in a header

##### Reason

It is almost always a bug to mention an unnamed namespace in a header file.

##### Example

    ???

##### Enforcement

* Flag any use of an anonymous namespace in a header file.

### <span id="Rs-unnamed2"></span>SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities

##### Reason

Nothing external can depend on an entity in a nested unnamed namespace.
Consider putting every definition in an implementation source file in an unnamed namespace unless that is defining an "external/exported" entity.

##### Example

An API class and its members can't live in an unnamed namespace; but any "helper" class or function that is defined in an implementation source file should be at an unnamed namespace scope.

    ???

##### Enforcement

* ???

# <span id="S-stdlib"></span>SL: The Standard Library

Using only the bare language, every task is tedious (in any language).
Using a suitable library any task can be reasonably simple.

The standard library has steadily grown over the years.
Its description in the standard is now larger than that of the language features.
So, it is likely that this library section of the guidelines will eventually grow in size to equal or exceed all the rest.

<< ??? We need another level of rule numbering ??? >>

C++ Standard library component summary:

* [SL.con: Containers](#SS-con)
* [SL.str: String](#SS-string)
* [SL.io: Iostream](#SS-io)
* [SL.regex: Regex](#SS-regex)
* [SL.chrono: Time](#SS-chrono)
* [SL.C: The C standard library](#SS-clib)

Standard-library rule summary:

* [SL.1: Use libraries wherever possible](#Rsl-lib)
* [SL.2: Prefer the standard library to other libraries](#Rsl-sl)
* ???

### <span id="Rsl-lib"></span>SL.1:  Use libraries wherever possible

##### Reason

Save time. Don't re-invent the wheel.
Don't replicate the work of others.
Benefit from other people's work when they make improvements.
Help other people when you make improvements.

### <span id="Rsl-sl"></span>SL.2: Prefer the standard library to other libraries

##### Reason

More people know the standard library.
It is more likely to be stable, well-maintained, and widely available than your own code or most other libraries.

## <span id="SS-con"></span>SL.con: Containers

???

Container rule summary:

* [SL.con.1: Prefer using STL `array` or `vector` instead of a C array](#Rsl-arrays)
* [SL.con.2: Prefer using STL `vector` by default unless you have a reason to use a different container](#Rsl-vector)
*  ???

### <span id="Rsl-arrays"></span>SL.con.1: Prefer using STL `array` or `vector` instead of a C array

##### Reason

C arrays are less safe, and have no advantages over `array` and `vector`.
For a fixed-length array, use `std::array`, which does not degenerate to a pointer when passed to a function and does know its size.
Also, like a built-in array, a stack-allocated `std::array` keeps its elements on the stack.
For a variable-length array, use `std::vector`, which additionally can change its size and handles memory allocation.

##### Example

    int v[SIZE];                        // BAD

    std::array<int, SIZE> w;             // ok

##### Example

    int* v = new int[initial_size];     // BAD, owning raw pointer
    delete[] v;                         // BAD, manual delete

    std::vector<int> w(initial_size);   // ok

##### Enforcement

* Flag declaration of a C array inside a function or class that also declares an STL container (to avoid excessive noisy warnings on legacy non-STL code). To fix: At least change the C array to a `std::array`.

### <span id="Rsl-vector"></span>SL.con.2: Prefer using STL `vector` by default unless you have a reason to use a different container

##### Reason

`vector` and `array` are the only standard containers that offer the fastest general-purpose access (random access, including being vectorization-friendly), the fastest default access pattern (begin-to-end or end-to-begin is prefetcher-friendly), and the lowest space overhead (contiguous layout has zero per-element overhead, which is cache-friendly).
Usually you need to add and remove elements from the container, so use `vector` by default; if you don't need to modify the container's size, use `array`.

Even when other containers seem more suited, such a `map` for O(log N) lookup performance or a `list` for efficient insertion in the middle, a `vector` will usually still perform better for containers up to a few KB in size.

##### Note

`string` should not be used as a container of individual characters. A `string` is a textual string; if you want a container of characters, use `vector</*char_type*/>` or `array</*char_type*/>` instead.

##### Exceptions

If you have a good reason to use another container, use that instead. For example:

* If `vector` suits your needs but you don't need the container to be variable size, use `array` instead.

* If you want a dictionary-style lookup container that guarantees O(K) or O(log N) lookups, the container will be larger (more than a few KB) and you perform frequent inserts so that the overhead of maintaining a sorted `vector` is infeasible, go ahead and use an `unordered_map` or `map` instead.

##### Enforcement

* Flag a `vector` whose size never changes after construction (such as because it's `const` or because no non-`const` functions are called on it). To fix: Use an `array` instead.

## <span id="SS-string"></span>SL.str: String

???

## <span id="SS-io"></span>SL.io: Iostream

???

Iostream rule summary:

* [SL.io.1: Use character-level input only when you have to](#Rio-low)
* [SL.io.2: When reading, always consider ill-formed input](#Rio-validate)
* [???](#???)
* [SL.io.50: Avoid `endl`](#Rio-endl)
* [???](#???)

### <span id="Rio-low"></span>SL.io.1: Use character-level input only when you have to

???

### <span id="Rio-validate"></span>SL.io.2: When reading, always consider ill-formed input

???

### <span id="Rio-endl"></span>SL.io.50: Avoid `endl`

### Reason

The `endl` manipulator is mostly equivalent to `'\n'` and `"\n"`;
as most commonly used it simply slows down output by doing redundant `flush()`s.
This slowdown can be significant compared to `printf`-style output.

##### Example

    cout << "Hello, World!" << endl;    // two output operations and a flush
    cout << "Hello, World!\n";          // one output operation and no flush

##### Note

For `cin`/`cout` (and equivalent) interaction, there is no reason to flush; that's done automatically.
For writing to a file, there is rarely a need to `flush`.

##### Note

Apart from the (occasionally important) issue of performance,
the choice between `'\n'` and `endl` is almost completely aesthetic.

## <span id="SS-regex"></span>SL.regex: Regex

???

## <span id="SS-chrono"></span>SL.chrono: Time

???

## <span id="SS-clib"></span>SL.C: The C standard library

???

C standard library rule summary:

* [???](#???)
* [???](#???)
* [???](#???)


# <span id="S-A"></span>A: Architectural Ideas

This section contains ideas about higher-level architectural ideas and libraries.

Architectural rule summary:

* [A.1 Separate stable from less stable part of code](#Ra-stable)
* [A.2 Express potentially reusable parts as a library](#Ra-lib)
* [A.4 There should be no cycles among libraries](#?Ra-dag)
* [???](#???)
* [???](#???)
* [???](#???)
* [???](#???)
* [???](#???)
* [???](#???)

### <span id="Ra-stable"></span>A.1 Separate stable from less stable part of code

???

### <span id="Ra-lib"></span>A.2 Express potentially reusable parts as a library

##### Reason

##### Note

A library is a collection of declarations and definitions maintained, documented, and shipped together.
A library could be a set of headers (a "header only library") or a set of headers plus a set of object files.
A library can be statically or dynamically linked into a program, or it may be `#included`


### <span id="Ra-dag"></span>A.4 There should be no cycles among libraries

##### Reason

* A cycle implies complication of the build process.
* Cycles are hard to understand and may introduce indeterminism (unspecified behavior).

##### Note

A library can contain cyclic references in the definition of its components.
For example:

    ???

However, a library should not depend on another that depends on it.


# <span id="S-not"></span>NR: Non-Rules and myths

This section contains rules and guidelines that are popular somewhere, but that we deliberately don't recommend.
We know full well that there have been times and places where these rules made sense, and we have used them ourselves at times.
However, in the context of the styles of programming we recommend and support with the guidelines, these "non-rules" would do harm.

Even today, there can be contexts where the rules make sense.
For example, lack of suitable tool support can make exceptions unsuitable in hard-real-time systems,
but please don't blindly trust "common wisdom" (e.g., unsupported statements about "efficiency");
such "wisdom" may be based on decades-old information or experienced from languages with very different properties than C++
(e.g., C or Java).

The positive arguments for alternatives to these non-rules are listed in the rules offered as "Alternatives".

Non-rule summary:

* [NR.1: Don't: All declarations should be at the top of a function](#Rnr-top)
* [NR.2: Don't: Have only a single `return`-statement in a function](#Rnr-single-return)
* [NR.3: Don't: Don't use exceptions](#Rnr-no-exceptions)
* [NR.4: Don't: Place each class declaration in its own source file](#Rnr-lots-of-files)
* [NR.5: Don't: Don't do substantive work in a constructor; instead use two-phase initialization](#Rnr-two-phase-init)
* [NR.6: Don't: Place all cleanup actions at the end of a function and `goto exit`](#Rnr-goto-exit)
* [NR.7: Don't: Make all data members `protected`](#Rnr-protected-data)
* ???

### <span id="Rnr-top"></span>NR.1: Don't: All declarations should be at the top of a function

##### Reason (not to follow this rule)

This rule is a legacy of old programming languages that didn't allow initialization of variables and constants after a statement.
This leads to longer programs and more errors caused by uninitialized and wrongly initialized variables.

##### Example, bad

    ???

The larger the distance between the uninitialized variable and its use, the larger the chance of a bug.
Fortunately, compilers catch many "used before set" errors.


##### Alternative

* [Always initialize an object](#Res-always)
* [ES.21: Don't introduce a variable (or constant) before you need to use it](#Res-introduce)

### <span id="Rnr-single-return"></span>NR.2: Don't: Have only a single `return`-statement in a function

##### Reason (not to follow this rule)

The single-return rule can lead to unnecessarily convoluted code and the introduction of extra state variables.
In particular, the single-return rule makes it harder to concentrate error checking at the top of a function.

##### Example

    template<class T>
    //  requires Number<T>
    string sign(T x)
    {
        if (x < 0)
            return "negative";
        else if (x > 0)
            return "positive";
        return "zero";
    }

to use a single return only we would have to do something like

    template<class T>
    //  requires Number<T>
    string sign(T x)        // bad
    {
        string res;
        if (x < 0)
            res = "negative";
        else if (x > 0)
            res = "positive";
        else
            res = "zero";
        return res;
    }

This is both longer and likely to be less efficient.
The larger and more complicated the function is, the more painful the workarounds get.
Of course many simple functions will naturally have just one `return` because of their simpler inherent logic.

##### Example

    int index(const char* p)
    {
        if (p == nullptr) return -1;  // error indicator: alternatively "throw nullptr_error{}"
        // ... do a lookup to find the index for p
        return i;
    }

If we applied the rule, we'd get something like

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

Note that we (deliberately) violated the rule against uninitialized variables because this style commonly leads to that.
Also, this style is a temptation to use the [goto exit](#Rnr-goto-exit) non-rule.

##### Alternative

* Keep functions short and simple
* Feel free to use multiple `return` statements (and to throw exceptions).

### <span id="Rnr-no-exceptions"></span>NR.3: Don't: Don't use exceptions

##### Reason (not to follow this rule)

There seem to be three main reasons given for this non-rule:

* exceptions are inefficient
* exceptions lead to leaks and errors
* exception performance is not predictable

There is no way we can settle this issue to the satisfaction of everybody.
After all, the discussions about exceptions have been going on for 40+ years.
Some languages cannot be used without exceptions, but others do not support them.
This leads to strong traditions for the use and non-use of exceptions, and to heated debates.

However, we can briefly outline why we consider exceptions the best alternative for general-purpose programming
and in the context of these guidelines.
Simple arguments for and against are often inconclusive.
There are specialized applications where exceptions indeed can be inappropriate
(e.g., hard-real time systems without support for reliable estimates of the cost of handling an exception).

Consider the major objections to exceptions in turn

* Exceptions are inefficient:
Compared to what?
When comparing make sure that the same set of errors are handled and that they are handled equivalently.
In particular, do not compare a program that immediately terminate on seeing an error with a program
that carefully cleans up resources before logging an error.
Yes, some systems have poor exception handling implementations; sometimes, such implementations force us to use
other error-handling approaches, but that's not a fundamental problem with exceptions.
When using an efficiency argument - in any context - be careful that you have good data that actually provides
insight into the problem under discussion.
* Exceptions lead to leaks and errors.
They do not.
If your program is a rat's nest of pointers without an overall strategy for resource management,
you have a problem whatever you do.
If your system consists of a million lines of such code,
you probably will not be able to use exceptions,
but that's a problem with excessive and undisciplined pointer use, rather than with exceptions.
In our opinion, you need RAII to make exception-based error handling simple and safe -- simpler and safer than alternatives.
* Exception performance is not predictable
If you are in a hard-real-time system where you must guarantee completion of a task in a given time,
you need tools to back up such guarantees.
As far as we know such tools are not available (at least not to most programmers).

Many, possibly most, problems with exceptions stem from historical needs to interact with messy old code.

The fundamental arguments for the use of exceptions are

* They clearly separates error return from ordinary return
* They cannot be forgotten or ignored
* They can be used systematically

Remember

* Exceptions are for reporting errors (in C++; other languages can have different uses for exceptions).
* Exceptions are not for errors that can be handled locally.
* Don't try to catch every exception in every function (that's tedious, clumsy, and leads to slow code).
* Exceptions are not for errors that require instant termination of a module/system after a non-recoverable error.

##### Example

    ???

##### Alternative

* [RAII](#Re-raii)
* Contracts/assertions: Use GSL's `Expects` and `Ensures` (until we get language support for contracts)

### <span id="Rnr-lots-of-files"></span>NR.4: Don't: Place each class declaration in its own source file

##### Reason (not to follow this rule)

The resulting number of files are hard to manage and can slow down compilation.
Individual classes are rarely a good logical unit of maintenance and distribution.

##### Example

    ???

##### Alternative

* Use namespaces containing logically cohesive sets of classes and functions.

### <span id="Rnr-two-phase-init"></span>NR.5: Don't: Don't do substantive work in a constructor; instead use two-phase initialization

##### Reason (not to follow this rule)

Following this rule leads to weaker invariants,
more complicated code (having to deal with semi-constructed objects),
and errors (when we didn't deal correctly with semi-constructed objects consistently).

##### Example

    ???

##### Alternative

* Always establish a class invariant in a constructor.
* Don't define an object before it is needed.

### <span id="Rnr-goto-exit"></span>NR.6: Don't: Place all cleanup actions at the end of a function and `goto exit`

##### Reason (not to follow this rule)

`goto` is error-prone.
This technique is a pre-exception technique for RAII-like resource and error handling.

##### Example, bad

    void do_something(int n)
    {
        if (n < 100) goto exit;
        // ...
        int* p = (int*) malloc(n);
        // ...
        if (some_ error) goto_exit;
        // ...
    exit:
        free(p);
    }

and spot the bug.

##### Alternative

* Use exceptions and [RAII](#Re-raii)
* for non-RAII resources, use [`finally`](#Re-finally).

### <span id="Rnr-protected-data"></span>NR.7: Don't: Make all data members `protected`

##### Reason (not to follow this rule)

`protected` data is a source of errors.
`protected` data can be manipulated from an unbounded amount of code in various places.
`protected` data is the class hierarchy equivalent to global data.

##### Example

    ???

##### Alternative

* [Make member data `public` or (preferably) `private`](#Rh-protected)

