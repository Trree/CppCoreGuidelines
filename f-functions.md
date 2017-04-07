# F: Functions {#f-functions}

A function specifies an action or a computation that takes the system from one consistent state to the next. It is the fundamental building block of programs.

It should be possible to name a function meaningfully, to specify the requirements of its argument, and clearly state the relationship between the arguments and the result. An implementation is not a specification. Try to think about what a function does as well as about how it does it. Functions are the most critical part in most interfaces, so see the interface rules.

Function rule summary:

Function definition rules:

* [F.1: “Package” meaningful operations as carefully named functions](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-package)
* [F.2: A function should perform a single logical operation](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-logical)
* [F.3: Keep functions short and simple](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-single)
* [F.4: If a function may have to be evaluated at compile time, declare it`constexpr`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-constexpr)
* [F.5: If a function is very small and time-critical, declare it inline](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-inline)
* [F.6: If your function may not throw, declare it`noexcept`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-noexcept)
* [F.7: For general use, take`T*`or`T&`arguments rather than smart pointers](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-smart)
* [F.8: Prefer pure functions](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-pure)
* [F.9: Unused parameters should be unnamed](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-unused)

Parameter passing expression rules:

* [F.15: Prefer simple and conventional ways of passing information](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-conventional)
* [F.16: For “in” parameters, pass cheaply-copied types by value and others by reference to`const`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-in)
* [F.17: For “in-out” parameters, pass by reference to non-`const`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-inout)
* [F.18: For “consume” parameters, pass by`X&&`and`std::move`the parameter](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-consume)
* [F.19: For “forward” parameters, pass by`TP&&`and only`std::forward`the parameter](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-forward)
* [F.20: For “out” output values, prefer return values to output parameters](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out)
* [F.21: To return multiple “out” values, prefer returning a tuple or struct](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out-multi)
* [F.60: Prefer`T*`over`T&`when “no argument” is a valid option](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-ptr-ref)

Parameter passing semantic rules:

* [F.22: Use`T*`or`owner<T*>`or a smart pointer to designate a single object](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-ptr)
* [F.23: Use a`not_null<T>`to indicate “null” is not a valid value](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-nullptr)
* [F.24: Use a`span<T>`or a`span_p<T>`to designate a half-open sequence](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-range)
* [F.25: Use a`zstring`or a`not_null<zstring>`to designate a C-style string](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-string)
* [F.26: Use a`unique_ptr<T>`to transfer ownership where a pointer is needed](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-unique_ptr)
* [F.27: Use a`shared_ptr<T>`to share ownership](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-shared_ptr)

Value return semantic rules:

* [F.42: Return a`T*`to indicate a position \(only\)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-return-ptr)
* [F.43: Never \(directly or indirectly\) return a pointer or a reference to a local object](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-dangle)
* [F.44: Return a`T&`when copy is undesirable and “returning no object” isn’t an option](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-return-ref)
* [F.45: Don’t return a`T&&`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-return-ref-ref)
* [F.46:`int`is the return type for`main()`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-main)
* [F.47: Return`T&`from assignment operators.](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-assignment-op)

Other function rules:

* [F.50: Use a lambda when a function won’t do \(to capture local variables, or to write a local function\)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-capture-vs-overload)
* [F.51: Where there is a choice, prefer default arguments over overloading](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-default-args)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-value-capture)
* [F.54: If you capture`this`, capture all variables explicitly \(no default capture\)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-this-capture)

Functions have strong similarities to lambdas and function objects so see also Section ???.

## F.def: Function definitions {#fdef-function-definitions}

A function definition is a function declaration that also specifies the function’s implementation, the function body.

### F.1: “Package” meaningful operations as carefully named functions {#f1-package-meaningful-operations-as-carefully-named-functions}

##### Reason {#reason-31}

Factoring out common code makes code more readable, more likely to be reused, and limit errors from complex code. If something is a well-specified action, separate it out from its surrounding code and give it a name.

##### Example, don’t {#example-dont-1}

```
void read_and_print(istream
&
 is)    // read and print an int
{
    int x;
    if (is 
>
>
 x)
        cout 
<
<
 "the int is " 
<
<
 x 
<
<
 '\n';
    else
        cerr 
<
<
 "no int on input\n";
}
```

Almost everything is wrong with`read_and_print`. It reads, it writes \(to a fixed`ostream`\), it writes error messages \(to a fixed`ostream`\), it handles only`int`s. There is nothing to reuse, logically separate operations are intermingled and local variables are in scope after the end of their logical use. For a tiny example, this looks OK, but if the input operation, the output operation, and the error handling had been more complicated the tangled mess could become hard to understand.

##### Note {#note-39}

If you write a non-trivial lambda that potentially can be used in more than one place, give it a name by assigning it to a \(usually non-local\) variable.

##### Example {#example-29}

```
sort(a, b, [](T x, T y) { return x.rank() 
<
 y.rank() 
&
&
 x.value() 
<
 y.value(); });
```

Naming that lambda breaks up the expression into its logical parts and provides a strong hint to the meaning of the lambda.

```
auto lessT = [](T x, T y) { return x.rank() 
<
 y.rank() 
&
&
 x.value() 
<
 y.value(); };

sort(a, b, lessT);
find_if(a, b, lessT);
```

The shortest code is not always the best for performance or maintainability.

##### Exception {#exception-7}

Loop bodies, including lambdas used as loop bodies, rarely need to be named. However, large loop bodies \(e.g., dozens of lines or dozens of pages\) can be a problem. The rule[Keep functions short](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-single)implies “Keep loop bodies short.” Similarly, lambdas used as callback arguments are sometimes non-trivial, yet unlikely to be re-usable.

##### Enforcement {#enforcement-28}

* See
  [Keep functions short](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-single)
* Flag identical and very similar lambdas used in different places.

### F.2: A function should perform a single logical operation {#f2-a-function-should-perform-a-single-logical-operation}

##### Reason {#reason-32}

A function that performs a single operation is simpler to understand, test, and reuse.

##### Example {#example-30}

Consider:

```
void read_and_print()    // bad
{
    int x;
    cin 
>
>
 x;
    // check for errors
    cout 
<
<
 x 
<
<
 "\n";
}
```

This is a monolith that is tied to a specific input and will never find another \(different\) use. Instead, break functions up into suitable logical parts and parameterize:

```
int read(istream
&
 is)    // better
{
    int x;
    is 
>
>
 x;
    // check for errors
    return x;
}

void print(ostream
&
 os, int x)
{
    os 
<
<
 x 
<
<
 "\n";
}
```

These can now be combined where needed:

```
void read_and_print()
{
    auto x = read(cin);
    print(cout, x);
}
```

If there was a need, we could further templatize`read()`and`print()`on the data type, the I/O mechanism, the response to errors, etc. Example:

```
auto read = [](auto
&
 input, auto
&
 value)    // better
{
    input 
>
>
 value;
    // check for errors
};

auto print(auto
&
 output, const auto
&
 value)
{
    output 
<
<
 value 
<
<
 "\n";
}
```

##### Enforcement {#enforcement-29}

* Consider functions with more than one “out” parameter suspicious. Use return values instead, including
  `tuple`
  for multiple return values.
* Consider “large” functions that don’t fit on one editor screen suspicious. Consider factoring such a function into smaller well-named suboperations.
* Consider functions with 7 or more parameters suspicious.

### F.3: Keep functions short and simple {#f3-keep-functions-short-and-simple}

##### Reason {#reason-33}

Large functions are hard to read, more likely to contain complex code, and more likely to have variables in larger than minimal scopes. Functions with complex control structures are more likely to be long and more likely to hide logical errors

##### Example {#example-31}

Consider:

```
double simpleFunc(double val, int flag1, int flag2)
    // simpleFunc: takes a value and calculates the expected ASIC output,
    // given the two mode flags.
{
    double intermediate;
    if (flag1 
>
 0) {
        intermediate = func1(val);
        if (flag2 % 2)
             intermediate = sqrt(intermediate);
    }
    else if (flag1 == -1) {
        intermediate = func1(-val);
        if (flag2 % 2)
             intermediate = sqrt(-intermediate);
        flag1 = -flag1;
    }
    if (abs(flag2) 
>
 10) {
        intermediate = func2(intermediate);
    }
    switch (flag2 / 10) {
        case 1: if (flag1 == -1) return finalize(intermediate, 1.171);
                break;
        case 2: return finalize(intermediate, 13.1);
        default: break;
    }
    return finalize(intermediate, 0.);
}
```

This is too complex \(and long\). How would you know if all possible alternatives have been correctly handled? Yes, it breaks other rules also.

We can refactor:

```
double func1_muon(double val, int flag)
{
    // ???
}

double funct1_tau(double val, int flag1, int flag2)
{
    // ???
}

double simpleFunc(double val, int flag1, int flag2)
    // simpleFunc: takes a value and calculates the expected ASIC output,
    // given the two mode flags.
{
    if (flag1 
>
 0)
        return func1_muon(val, flag2);
    if (flag1 == -1)
        // handled by func1_tau: flag1 = -flag1;
        return func1_tau(-val, flag1, flag2);
    return 0.;
}
```

##### Note {#note-40}

“It doesn’t fit on a screen” is often a good practical definition of “far too large.” One-to-five-line functions should be considered normal.

##### Note {#note-41}

Break large functions up into smaller cohesive and named functions. Small simple functions are easily inlined where the cost of a function call is significant.

##### Enforcement {#enforcement-30}

* Flag functions that do not “fit on a screen.” How big is a screen? Try 60 lines by 140 characters; that’s roughly the maximum that’s comfortable for a book page.
* Flag functions that are too complex. How complex is too complex? You could use cyclomatic complexity. Try “more than 10 logical path through.” Count a simple switch as one path.

### F.4: If a function may have to be evaluated at compile time, declare it`constexpr` {#f4-if-a-function-may-have-to-be-evaluated-at-compile-time-declare-it-constexpr}

##### Reason {#reason-34}

`constexpr`is needed to tell the compiler to allow compile-time evaluation.

##### Example {#example-32}

The \(in\)famous factorial:

```
constexpr int fac(int n)
{
    constexpr int max_exp = 17;      // constexpr enables max_exp to be used in Expects
    Expects(0 
<
= n 
&
&
 n 
<
 max_exp);  // prevent silliness and overflow
    int x = 1;
    for (int i = 2; i 
<
= n; ++i) x *= i;
    return x;
}
```

This is C++14. For C++11, use a recursive formulation of`fac()`.

##### Note {#note-42}

`constexpr`does not guarantee compile-time evaluation; it just guarantees that the function can be evaluated at compile time for constant expression arguments if the programmer requires it or the compiler decides to do so to optimize.

```
constexpr int min(int x, int y) { return x 
<
 y ? x : y; }

void test(int v)
{
    int m1 = min(-1, 2);            // probably compile-time evaluation
    constexpr int m2 = min(-1, 2);  // compile-time evaluation
    int m3 = min(-1, v);            // run-time evaluation
    constexpr int m4 = min(-1, v);  // error: cannot evaluate at compile-time
}
```

##### Note {#note-43}

`constexpr`functions are pure: they can have no side effects.

```
int dcount = 0;
constexpr int double(int v)
{
    ++dcount;   // error: attempted side effect from constexpr function
    return v + v;
}
```

This is usually a very good thing.

When given a non-constant argument, a`constexpr`function can throw. If you consider exiting by throwing a side-effect, a`constexpr`function isn’t completely pure; if not, this is not an issue. ??? A question for the committee: can a constructor for an exception thrown by a`constexpr`function modify state? “No” would be a nice answer that matches most practice.

##### Note {#note-44}

Don’t try to make all functions`constexpr`. Most computation is best done at run time.

##### Note {#note-45}

Any API that may eventually depend on high-level runtime configuration or business logic should not be made`constexpr`. Such customization can not be evaluated by the compiler, and any`constexpr`functions that depended upon that API would have to be refactored or drop`constexpr`.

##### Enforcement {#enforcement-31}

Impossible and unnecessary. The compiler gives an error if a non-`constexpr`function is called where a constant is required.

### F.5: If a function is very small and time-critical, declare it`inline` {#f5-if-a-function-is-very-small-and-time-critical-declare-it-inline}

##### Reason {#reason-35}

Some optimizers are good at inlining without hints from the programmer, but don’t rely on it. Measure! Over the last 40 years or so, we have been promised compilers that can inline better than humans without hints from humans. We are still waiting. Specifying`inline`encourages the compiler to do a better job.

##### Example {#example-33}

```
inline string cat(const string
&
 s, const string
&
 s2) { return s + s2; }
```

##### Exception {#exception-8}

Do not put an`inline`function in what is meant to be a stable interface unless you are certain that it will not change. An inline function is part of the ABI.

##### Note {#note-46}

`constexpr`implies`inline`.

##### Note {#note-47}

Member functions defined in-class are`inline`by default.

##### Exception {#exception-9}

Template functions \(incl. template member functions\) must be in headers and therefore inline.

##### Enforcement {#enforcement-32}

Flag`inline`functions that are more than three statements and could have been declared out of line \(such as class member functions\).

### F.6: If your function may not throw, declare it`noexcept` {#f6-if-your-function-may-not-throw-declare-it-noexcept}

##### Reason {#reason-36}

If an exception is not supposed to be thrown, the program cannot be assumed to cope with the error and should be terminated as soon as possible. Declaring a function`noexcept`helps optimizers by reducing the number of alternative execution paths. It also speeds up the exit after failure.

##### Example {#example-34}

Put`noexcept`on every function written completely in C or in any other language without exceptions. The C++ standard library does that implicitly for all functions in the C standard library.

##### Note {#note-48}

`constexpr`functions can throw when evaluated at run time, so you may need`noexcept`for some of those.

##### Example {#example-35}

You can use`noexcept`even on functions that can throw:

```
vector
<
string
>
 collect(istream
&
 is) noexcept
{
    vector
<
string
>
 res;
    for (string s; is 
>
>
 s;)
        res.push_back(s);
    return res;
}
```

If`collect()`runs out of memory, the program crashes. Unless the program is crafted to survive memory exhaustion, that may be just the right thing to do;`terminate()`may generate suitable error log information \(but after memory runs out it is hard to do anything clever\).

##### Note {#note-49}

You must be aware of the execution environment that your code is running when deciding whether to tag a function`noexcept`, especially because of the issue of throwing and allocation. Code that is intended to be perfectly general \(like the standard library and other utility code of that sort\) needs to support environments where a`bad_alloc`exception may be handled meaningfully. However, most programs and execution environments cannot meaningfully handle a failure to allocate, and aborting the program is the cleanest and simplest response to an allocation failure in those cases. If you know that your application code cannot respond to an allocation failure, it may be appropriate to add`noexcept`even on functions that allocate.

Put another way: In most programs, most functions can throw \(e.g., because they use`new`, call functions that do, or use library functions that reports failure by throwing\), so don’t just sprinkle`noexcept`all over the place without considering whether the possible exceptions can be handled.

`noexcept`is most useful \(and most clearly correct\) for frequently used, low-level functions.

##### Note {#note-50}

Destructors,`swap`functions, move operations, and default constructors should never throw.

##### Enforcement {#enforcement-33}

* Flag functions that are not
  `noexcept`
  , yet cannot throw.
* Flag throwing
  `swap`
  ,
  `move`
  , destructors, and default constructors.



### F.7: For general use, take`T*`or`T&`arguments rather than smart pointers {#f7-for-general-use-take-t-or-t-arguments-rather-than-smart-pointers}

##### Reason {#reason-37}

Passing a smart pointer transfers or shares ownership and should only be used when ownership semantics are intended \(see[R.30](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-smartptrparam)\). Passing by smart pointer restricts the use of a function to callers that use smart pointers. Passing a shared smart pointer \(e.g.,`std::shared_ptr`\) implies a run-time cost.

##### Example {#example-36}

```
// accepts any int*
void f(int*);

// can only accept ints for which you want to transfer ownership
void g(unique_ptr
<
int
>
);

// can only accept ints for which you are willing to share ownership
void g(shared_ptr
<
int
>
);

// doesn't change ownership, but requires a particular ownership of the caller
void h(const unique_ptr
<
int
>
&
);

// accepts any int
void h(int
&
);

```

##### Example, bad {#example-bad-16}

```
// callee
void f(shared_ptr
<
widget
>
&
 w)
{
    // ...
    use(*w); // only use of w -- the lifetime is not used at all
    // ...
};

```

See further in[R.30](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-smartptrparam).

##### Note {#note-51}

We can catch dangling pointers statically, so we don’t need to rely on resource management to avoid violations from dangling pointers.

**See also**:[when to prefer`T*`and when to prefer`T&`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-ptr-ref).

**See also**: Discussion of[smart pointer use](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-summary-smartptrs).

##### Enforcement {#enforcement-34}

Flag a parameter of a smart pointer type \(a type that overloads`operator->`or`operator*`\) for which the ownership semantics are not used; that is

* copyable but never copied/moved from or movable but never moved
* and that is never modified or passed along to another function that could do so.

### F.8: Prefer pure functions {#f8-prefer-pure-functions}

##### Reason {#reason-38}

Pure functions are easier to reason about, sometimes easier to optimize \(and even parallelize\), and sometimes can be memoized.

##### Example {#example-37}

```
template
<
class T
>

auto square(T t) { return t * t; }

```

##### Note {#note-52}

`constexpr`functions are pure.

When given a non-constant argument, a`constexpr`function can throw. If you consider exiting by throwing a side-effect, a`constexpr`function isn’t completely pure; if not, this is not an issue. ??? A question for the committee: can a constructor for an exception thrown by a`constexpr`function modify state? “No” would be a nice answer that matches most practice.

##### Enforcement {#enforcement-35}

Not possible.

### F.9: Unused parameters should be unnamed {#f9-unused-parameters-should-be-unnamed}

##### Reason {#reason-39}

Readability. Suppression of unused parameter warnings.

##### Example {#example-38}

```
X* find(map
<
Blob
>
&
 m, const string
&
 s, Hint);   // once upon a time, a hint was used

```

##### Note {#note-53}

Allowing parameters to be unnamed was introduced in the early 1980 to address this problem.

##### Enforcement {#enforcement-36}

Flag named unused parameters.

## F.call: Parameter passing {#fcall-parameter-passing}

There are a variety of ways to pass parameters to a function and to return values.

### F.15: Prefer simple and conventional ways of passing information {#f15-prefer-simple-and-conventional-ways-of-passing-information}

##### Reason {#reason-40}

Using “unusual and clever” techniques causes surprises, slows understanding by other programmers, and encourages bugs. If you really feel the need for an optimization beyond the common techniques, measure to ensure that it really is an improvement, and document/comment because the improvement may not be portable.

The following tables summarize the advice in the following Guidelines, F.16-21.

Normal parameter passing:

![](http://isocpp.github.io/CppCoreGuidelines/param-passing-normal.png "Normal parameter passing table")

Advanced parameter passing:

![](http://isocpp.github.io/CppCoreGuidelines/param-passing-advanced.png "Advanced parameter passing table")

Use the advanced techniques only after demonstrating need, and document that need in a comment.

### F.16: For “in” parameters, pass cheaply-copied types by value and others by reference to`const` {#f16-for-in-parameters-pass-cheaply-copied-types-by-value-and-others-by-reference-to-const}

##### Reason {#reason-41}

Both let the caller know that a function will not modify the argument, and both allow initialization by rvalues.

What is “cheap to copy” depends on the machine architecture, but two or three words \(doubles, pointers, references\) are usually best passed by value. When copying is cheap, nothing beats the simplicity and safety of copying, and for small objects \(up to two or three words\) it is also faster than passing by reference because it does not require an extra indirection to access from the function.

##### Example {#example-39}

```
void f1(const string
&
 s);  // OK: pass by reference to const; always cheap

void f2(string s);         // bad: potentially expensive

void f3(int x);            // OK: Unbeatable

void f4(const int
&
 x);     // bad: overhead on access in f4()

```

For advanced uses \(only\), where you really need to optimize for rvalues passed to “input-only” parameters:

* If the function is going to unconditionally move from the argument, take it by
  `&`
  `&`
  . See
  [F.18](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-consume)
  .
* If the function is going to keep a copy of the argument, in addition to passing by
  `const`
  `&`
  \(for lvalues\), add an overload that passes the parameter by
  `&`
  `&`
  \(for rvalues\) and in the body
  `std::move`
  s it to its destination. Essentially this overloads a “consume”; see
  [F.18](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-consume)
  .
* In special cases, such as multiple “input + copy” parameters, consider using perfect forwarding. See
  [F.19](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-forward)
  .

##### Example {#example-40}

```
int multiply(int, int); // just input ints, pass by value

// suffix is input-only but not as cheap as an int, pass by const
&

string
&
 concatenate(string
&
, const string
&
 suffix);

void sink(unique_ptr
<
widget
>
);  // input only, and consumes the widget

```

Avoid “esoteric techniques” such as:

* Passing arguments as
  `T`
  `&`
  `&`
  “for efficiency”. Most rumors about performance advantages from passing by
  `&`
  `&`
  are false or brittle \(but see
  [F.25](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-pass-ref-move)
  .\)
* Returning
  `const T`
  `&`
  from assignments and similar operations \(see
  [F.47](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-assignment-op)
  .\)

##### Example {#example-41}

Assuming that`Matrix`has move operations \(possibly by keeping its elements in a`std::vector`\):

```
Matrix operator+(const Matrix
&
 a, const Matrix
&
 b)
{
    Matrix res;
    // ... fill res with the sum ...
    return res;
}

Matrix x = m1 + m2;  // move constructor

y = m3 + m3;         // move assignment

```

##### Notes {#notes}

The return value optimization doesn’t handle the assignment case, but the move assignment does.

A reference may be assumed to refer to a valid object \(language rule\). There is no \(legitimate\) “null reference.” If you need the notion of an optional value, use a pointer,`std::optional`, or a special value used to denote “no value.”

##### Enforcement {#enforcement-37}

* \(Simple\) \(\(Foundation\)\) Warn when a parameter being passed by value has a size greater than
  `4 * sizeof(int)`
  . Suggest using a reference to
  `const`
  instead.
* \(Simple\) \(\(Foundation\)\) Warn when a
  `const`
  parameter being passed by reference has a size less than
  `3 * sizeof(int)`
  . Suggest passing by value instead.
* \(Simple\) \(\(Foundation\)\) Warn when a
  `const`
  parameter being passed by reference is
  `move`
  d.

### F.17: For “in-out” parameters, pass by reference to non-`const` {#f17-for-in-out-parameters-pass-by-reference-to-non-const}

##### Reason {#reason-42}

This makes it clear to callers that the object is assumed to be modified.

##### Example {#example-42}

```
void update(Record
&
 r);  // assume that update writes to r

```

##### Note {#note-54}

A`T&`argument can pass information into a function as well as well as out of it. Thus`T&`could be an in-out-parameter. That can in itself be a problem and a source of errors:

```
void f(string
&
 s)
{
    s = "New York";  // non-obvious error
}

void g()
{
    string buffer = ".................................";
    f(buffer);
    // ...
}

```

Here, the writer of`g()`is supplying a buffer for`f()`to fill, but`f()`simply replaces it \(at a somewhat higher cost than a simple copy of the characters\). A bad logic error can happen if the writer of`g()`incorrectly assumes the size of the`buffer`.

##### Enforcement {#enforcement-38}

* \(Moderate\) \(\(Foundation\)\) Warn about functions regarding reference to non-
  `const`
  parameters that do
  _not_
  write to them.
* \(Simple\) \(\(Foundation\)\) Warn when a non-
  `const`
  parameter being passed by reference is
  `move`
  d.

### F.18: For “consume” parameters, pass by`X&&`and`std::move`the parameter {#f18-for-consume-parameters-pass-by-x-and-stdmove-the-parameter}

##### Reason {#reason-43}

It’s efficient and eliminates bugs at the call site:`X&&`binds to rvalues, which requires an explicit`std::move`at the call site if passing an lvalue.

##### Example {#example-43}

```
void sink(vector
<
int
>
&
&
 v) {   // sink takes ownership of whatever the argument owned
    // usually there might be const accesses of v here
    store_somewhere(std::move(v));
    // usually no more use of v here; it is moved-from
}

```

Note that the`std::move(v)`makes it possible for`store_somewhere()`to leave`v`in a moved-from state.[That could be dangerous](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-semantic).

##### Exception {#exception-10}

Unique owner types that are move-only and cheap-to-move, such as`unique_ptr`, can also be passed by value which is simpler to write and achieves the same effect. Passing by value does generate one extra \(cheap\) move operation, but prefer simplicity and clarity first.

For example:

```
template 
<
class T
>

void sink(std::unique_ptr
<
T
>
 p) {
    // use p ... possibly std::move(p) onward somewhere else
}   // p gets destroyed

```

##### Enforcement {#enforcement-39}

* Flag all
  `X`
  `&`
  `&`
  parameters \(where
  `X`
  is not a template type parameter name\) where the function body uses them without
  `std::move`
  .
* Flag access to moved-from objects.
* Don’t conditionally move from objects

  




