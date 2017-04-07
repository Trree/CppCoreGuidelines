# NL: Naming and layout rules {#nl-naming-and-layout-rules}

Consistent naming and layout are helpful. If for no other reason because it minimizes “my style is better than your style” arguments. However, there are many, many, different styles around and people are passionate about them \(pro and con\). Also, most real-world projects includes code from many sources, so standardizing on a single style for all code is often impossible. We present a set of rules that you might use if you have no better ideas, but the real aim is consistency, rather than any particular rule set. IDEs and tools can help \(as well as hinder\).

Naming and layout rules:

* [NL.1: Don’t say in comments what can be clearly stated in code](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-comments)
* [NL.2: State intent in comments](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-comments-intent)
* [NL.3: Keep comments crisp](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-comments-crisp)
* [NL.4: Maintain a consistent indentation style](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-indent)
* [NL.5: Don’t encode type information in names](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-name-type)
* [NL.7: Make the length of a name roughly proportional to the length of its scope](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-name-length)
* [NL.8: Use a consistent naming style](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-name)
* [NL.9: Use`ALL_CAPS`for macro names only](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-all-caps)
* [NL.10: Avoid CamelCase](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-camel)
* [NL.15: Use spaces sparingly](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-space)
* [NL.16: Use a conventional class member declaration order](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-order)
* [NL.17: Use K&R-derived layout](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-knr)
* [NL.18: Use C++-style declarator layout](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-ptr)
* [NL.19: Avoid names that are easily misread](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-misread)
* [NL.20: Don’t place two statements on the same line](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-stmt)
* [NL.21: Declare one name \(only\) per declaration](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-dcl)
* [NL.25: Don’t use`void`as an argument type](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-void)
* [NL.26: Use conventional`const`notation](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-const)

Most of these rules are aesthetic and programmers hold strong opinions. IDEs also tend to have defaults and a range of alternatives. These rules are suggested defaults to follow unless you have reasons not to.

We have had comments to the effect that naming and layout are so personal and/or arbitrary that we should not try to “legislate” them. We are not “legislating” \(see the previous paragraph\). However, we have had many requests for a set of naming and layout conventions to use when there are no external constraints.

More specific and detailed rules are easier to enforce.

These rules bear a strong resemblance to the recommendations in the[PPP Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)written in support of Stroustrup’s[Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html).

### NL.1: Don’t say in comments what can be clearly stated in code {#nl1-dont-say-in-comments-what-can-be-clearly-stated-in-code}

##### Reason {#reason-403}

Compilers do not read comments. Comments are less precise than code. Comments are not updated as consistently as code.

##### Example, bad {#example-bad-139}

```
auto x = m * v1 + vv;   // multiply m with v1 and add the result to vv

```

##### Enforcement {#enforcement-367}

Build an AI program that interprets colloquial English text and see if what is said could be better expressed in C++.

### NL.2: State intent in comments {#nl2-state-intent-in-comments}

##### Reason {#reason-404}

Code says what is done, not what is supposed to be done. Often intent can be stated more clearly and concisely than the implementation.

##### Example {#example-342}

```
void stable_sort(Sortable
&
 c)
    // sort c in the order determined by 
<
, keep equal elements (as defined by ==) in
    // their original relative order
{
    // ... quite a few lines of non-trivial code ...
}

```

##### Note {#note-304}

If the comment and the code disagrees, both are likely to be wrong.

### NL.3: Keep comments crisp {#nl3-keep-comments-crisp}

##### Reason {#reason-405}

Verbosity slows down understanding and makes the code harder to read by spreading it around in the source file.

##### Note {#note-305}

Use intelligible English. I may be fluent in Danish, but most programmers are not; the maintainers of my code may not be. Avoid SMS lingo and watch your grammar, punctuation, and capitalization. Aim for professionalism, not “cool.”

##### Enforcement {#enforcement-368}

not possible.

### NL.4: Maintain a consistent indentation style {#nl4-maintain-a-consistent-indentation-style}

##### Reason {#reason-406}

Readability. Avoidance of “silly mistakes.”

##### Example, bad {#example-bad-140}

```
int i;
for (i = 0; i 
<
 max; ++i); // bug waiting to happen
if (i == j)
    return i;

```

##### Note {#note-306}

Always indenting the statement after`if (...)`,`for (...)`, and`while (...)`is usually a good idea:

```
if (i 
<
 0) error("negative argument");

if (i 
<
 0)
    error("negative argument");

```

##### Enforcement {#enforcement-369}

Use a tool.

### NL.5 Don’t encode type information in names {#nl5-dont-encode-type-information-in-names}

##### Rationale {#rationale}

If names reflect types rather than functionality, it becomes hard to change the types used to provide that functionality. Also, if the type of a variable is changed, code using it will have to be modified. Minimize unintentional conversions.

##### Example, bad {#example-bad-141}

```
void print_int(int i);
void print_string(const char*);

print_int(1);   // OK
print_int(x);   // conversion to int if x is a double

```

##### Note {#note-307}

Names with types encoded are either verbose or cryptic.

```
printS  // print a std::string
prints  // print a C-style string
printi  // print an int

```

PS. Hungarian notation is evil \(at least in a strongly statically-typed language\).

##### Note {#note-308}

Some styles distinguishes members from local variable, and/or from global variable.

```
struct S {
    int m_;
    S(int m) :m_{abs(m)} { }
};

```

This is not evil.

##### Note {#note-309}

Like C++, some styles distinguishes types from non-types. For example, by capitalizing type names, but not the names of functions and variables.

```
typename
<
typename T
>

class Hash_tbl {   // maps string to T
    // ...
};

Hash_tbl
<
int
>
 index;

```

This is not evil.

### NL.7: Make the length of a name roughly proportional to the length of its scope {#nl7-make-the-length-of-a-name-roughly-proportional-to-the-length-of-its-scope}

**Rationale**: The larger the scope the greater the chance of confusion and of an unintended name clash.

##### Example {#example-343}

```
double sqrt(double x);   // return the square root of x; x must be non-negative

int length(const char* p);  // return the number of characters in a zero-terminated C-style string

int length_of_string(const char zero_terminated_array_of_char[])    // bad: verbose

int g;      // bad: global variable with a cryptic name

int open;   // bad: global variable with a short, popular name

```

The use of`p`for pointer and`x`for a floating-point variable is conventional and non-confusing in a restricted scope.

##### Enforcement {#enforcement-370}

???

### NL.8: Use a consistent naming style {#nl8-use-a-consistent-naming-style}

**Rationale**: Consistence in naming and naming style increases readability.

##### Note {#note-310}

There are many styles and when you use multiple libraries, you can’t follow all their different conventions. Choose a “house style”, but leave “imported” libraries with their original style.

##### Example {#example-344}

ISO Standard, use lower case only and digits, separate words with underscores:

* `int`
* `vector`
* `my_map`

Avoid double underscores`__`.

##### Example {#example-345}

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf): ISO Standard, but with upper case used for your own types and concepts:

* `int`
* `vector`
* `My_map`

##### Example {#example-346}

CamelCase: capitalize each word in a multi-word identifier:

* `int`
* `vector`
* `MyMap`
* `myMap`

Some conventions capitalize the first letter, some don’t.

##### Note {#note-311}

Try to be consistent in your use of acronyms and lengths of identifiers:

```
int mtbf {12};
int mean_time_between_failures {12}; // make up your mind

```

##### Enforcement {#enforcement-371}

Would be possible except for the use of libraries with varying conventions.

### NL.9: Use`ALL_CAPS`for macro names only {#nl9-use-all_caps-for-macro-names-only}

##### Reason {#reason-407}

To avoid confusing macros with names that obey scope and type rules.

##### Example {#example-347}

```
void f()
{
    const int SIZE{1000};  // Bad, use 'size' instead
    int v[SIZE];
}

```

##### Note {#note-312}

This rule applies to non-macro symbolic constants:

```
enum bad { BAD, WORSE, HORRIBLE }; // BAD

```

##### Enforcement {#enforcement-372}

* Flag macros with lower-case letters
* Flag
  `ALL_CAPS`
  non-macro names

### NL.10: Avoid CamelCase {#nl10-avoid-camelcase}

##### Reason {#reason-408}

The use of underscores to separate parts of a name is the original C and C++ style and used in the C++ standard library. If you prefer CamelCase, you have to choose among different flavors of camelCase.

##### Note {#note-313}

This rule is a default to use only if you have a choice. Often, you don’t have a choice and must follow an established style for[consistency](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rl-name). The need for consistency beats personal taste.

##### Example {#example-348}

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf): ISO Standard, but with upper case used for your own types and concepts:

* `int`
* `vector`
* `My_map`

##### Enforcement {#enforcement-373}

Impossible.

### NL.15: Use spaces sparingly {#nl15-use-spaces-sparingly}

##### Reason {#reason-409}

Too much space makes the text larger and distracts.

##### Example, bad {#example-bad-142}

```
#include 
<
 map 
>


int main(int argc, char * argv [ ])
{
    // ...
}

```

##### Example {#example-349}

```
#include 
<
map
>


int main(int argc, char* argv[])
{
    // ...
}

```

##### Note {#note-314}

Some IDEs have their own opinions and add distracting space.

##### Note {#note-315}

We value well-placed whitespace as a significant help for readability. Just don’t overdo it.

### NL.16: Use a conventional class member declaration order {#nl16-use-a-conventional-class-member-declaration-order}

##### Reason {#reason-410}

A conventional order of members improves readability.

When declaring a class use the following order

* types: classes, enums, and aliases \(
  `using`
  \)
* constructors, assignments, destructor
* functions
* data

Use the`public`before`protected`before`private`order.

Private types and functions can be placed with private data.

Avoid multiple blocks of declarations of one access \(e.g.,`public`\) dispersed among blocks of declarations with different access \(e.g.`private`\).

##### Example {#example-350}

```
class X {
public:
    // interface
protected:
    // unchecked function for use by derived class implementations
private:
    // implementation details
};

```

##### Note {#note-316}

The use of macros to declare groups of members often violates any ordering rules. However, macros obscures what is being expressed anyway.

##### Enforcement {#enforcement-374}

Flag departures from the suggested order. There will be a lot of old code that doesn’t follow this rule.

### NL.17: Use K&R-derived layout {#nl17-use-kr-derived-layout}

##### Reason {#reason-411}

This is the original C and C++ layout. It preserves vertical space well. It distinguishes different language constructs \(such as functions and classes\) well.

##### Note {#note-317}

In the context of C++, this style is often called “Stroustrup”.

##### Example {#example-351}

```
struct Cable {
    int x;
    // ...
};

double foo(int x)
{
    if (0 
<
 x) {
        // ...
    }

    switch (x) {
        case 0:
            // ...
            break;
        case amazing:
            // ...
            break;
        default:
            // ...
            break;
    }

    if (0 
<
 x)
        ++x;

    if (x 
<
 0)
        something();
    else
        something_else();

    return some_value;
}

```

Note the space between`if`and`(`

##### Note {#note-318}

Use separate lines for each statement, the branches of an`if`, and the body of a`for`.

##### Note {#note-319}

The`{`for a`class`and a`struct`in_not_on a separate line, but the`{`for a function is.

##### Note {#note-320}

Capitalize the names of your user-defined types to distinguish them from standards-library types.

##### Note {#note-321}

Do not capitalize function names.

##### Enforcement {#enforcement-375}

If you want enforcement, use an IDE to reformat.

### NL.18: Use C++-style declarator layout {#nl18-use-c-style-declarator-layout}

##### Reason {#reason-412}

The C-style layout emphasizes use in expressions and grammar, whereas the C++-style emphasizes types. The use in expressions argument doesn’t hold for references.

##### Example {#example-352}

```
T
&
 operator[](size_t);   // OK
T 
&
operator[](size_t);   // just strange
T 
&
 operator[](size_t);   // undecided

```

##### Enforcement {#enforcement-376}

Impossible in the face of history.

### NL.19: Avoid names that are easily misread {#nl19-avoid-names-that-are-easily-misread}

##### Reason {#reason-413}

Readability. Not everyone has screens and printers that make it easy to distinguish all characters. We easily confuse similarly spelled and slightly misspelled words.

##### Example {#example-353}

```
int oO01lL = 6; // bad

int splunk = 7;
int splonk = 8; // bad: splunk and splonk are easily confused

```

##### Enforcement {#enforcement-377}

???

### NL.20: Don’t place two statements on the same line {#nl20-dont-place-two-statements-on-the-same-line}

##### Reason {#reason-414}

Readability. It is really easy to overlook a statement when there is more on a line.

##### Example {#example-354}

```
int x = 7; char* p = 29;    // don't
int x = 7; f(x);  ++x;      // don't

```

##### Enforcement {#enforcement-378}

Easy.

### NL.21: Declare one name \(only\) per declaration {#nl21-declare-one-name-only-per-declaration}

##### Reason {#reason-415}

Readability. Minimizing confusion with the declarator syntax.

##### Note {#note-322}

For details, see[ES.10](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-name-one).

### NL.25: Don’t use`void`as an argument type {#nl25-dont-use-void-as-an-argument-type}

##### Reason {#reason-416}

It’s verbose and only needed where C compatibility matters.

##### Example {#example-355}

```
void f(void);   // bad

void g();       // better

```

##### Note {#note-323}

Even Dennis Ritchie deemed`void f(void)`an abomination. You can make an argument for that abomination in C when function prototypes were rare so that banning:

```
int f();
f(1, 2, "weird but valid C89");   // hope that f() is defined int f(a, b, c) char* c; { /* ... */ }

```

would have caused major problems, but not in the 21st century and in C++.

### NL.26: Use conventional`const`notation {#nl26-use-conventional-const-notation}

##### Reason {#reason-417}

Conventional notation is more familiar to more programmers. Consistency in large code bases.

##### Example {#example-356}

```
const int x = 7;    // OK
int const y = 9;    // bad

const int *const p = nullptr;   // OK, constant pointer to constant int
int const *const p = nullptr;   // bad, constant pointer to constant int

```

##### Note {#note-324}

We are well aware that you could claim the “bad” examples more logical than the ones marked “OK”, but they also confuse more people, especially novices relying on teaching material using the far more common, conventional OK style.

As ever, remember that the aim of these naming and layout rules is consistency and that aesthetics vary immensely.

##### Enforcement {#enforcement-379}

Flag`const`used as a suffix for a type.

  


