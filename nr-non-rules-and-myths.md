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

  


