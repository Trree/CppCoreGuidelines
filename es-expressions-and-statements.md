# <a name="S-expr"></a>ES: Expressions and Statements

Expressions and statements are the lowest and most direct way of expressing actions and computation. Declarations in local scopes are statements.

For naming, commenting, and indentation rules, see [NL: Naming and layout](#S-naming).

General rules:

* [ES.1: Prefer the standard library to other libraries and to "handcrafted code"](#Res-lib)
* [ES.2: Prefer suitable abstractions to direct use of language features](#Res-abstr)

Declaration rules:

* [ES.5: Keep scopes small](#Res-scope)
* [ES.6: Declare names in for-statement initializers and conditions to limit scope](#Res-cond)
* [ES.7: Keep common and local names short, and keep uncommon and nonlocal names longer](#Res-name-length)
* [ES.8: Avoid similar-looking names](#Res-name-similar)
* [ES.9: Avoid `ALL_CAPS` names](#Res-not-CAPS)
* [ES.10: Declare one name (only) per declaration](#Res-name-one)
* [ES.11: Use `auto` to avoid redundant repetition of type names](#Res-auto)
* [ES.12: Do not reuse names in nested scopes](#Res-reuse)
* [ES.20: Always initialize an object](#Res-always)
* [ES.21: Don't introduce a variable (or constant) before you need to use it](#Res-introduce)
* [ES.22: Don't declare a variable until you have a value to initialize it with](#Res-init)
* [ES.23: Prefer the `{}`-initializer syntax](#Res-list)
* [ES.24: Use a `unique_ptr<T>` to hold pointers in code that may throw](#Res-unique)
* [ES.25: Declare an object `const` or `constexpr` unless you want to modify its value later on](#Res-const)
* [ES.26: Don't use a variable for two unrelated purposes](#Res-recycle)
* [ES.27: Use `std::array` or `stack_array` for arrays on the stack](#Res-stack)
* [ES.28: Use lambdas for complex initialization, especially of `const` variables](#Res-lambda-init)
* [ES.30: Don't use macros for program text manipulation](#Res-macros)
* [ES.31: Don't use macros for constants or "functions"](#Res-macros2)
* [ES.32: Use `ALL_CAPS` for all macro names](#Res-ALL_CAPS)
* [ES.33: If you must use macros, give them unique names](#Res-MACROS)
* [ES.34: Don't define a (C-style) variadic function](#Res-ellipses)

Expression rules:

* [ES.40: Avoid complicated expressions](#Res-complicated)
* [ES.41: If in doubt about operator precedence, parenthesize](#Res-parens)
* [ES.42: Keep use of pointers simple and straightforward](#Res-ptr)
* [ES.43: Avoid expressions with undefined order of evaluation](#Res-order)
* [ES.44: Don't depend on order of evaluation of function arguments](#Res-order-fct)
* [ES.45: Avoid narrowing conversions](#Res-narrowing)
* [ES.46: Avoid "magic constants"; use symbolic constants](#Res-magic)
* [ES.47: Use `nullptr` rather than `0` or `NULL`](#Res-nullptr)
* [ES.48: Avoid casts](#Res-casts)
* [ES.49: If you must use a cast, use a named cast](#Res-casts-named)
* [ES.50: Don't cast away `const`](#Res-casts-const)
* [ES.55: Avoid the need for range checking](#Res-range-checking)
* [ES.56: Write `std::move()` only when you need to explicitly move an object to another scope](#Res-move)
* [ES.60: Avoid `new` and `delete` outside resource management functions](#Res-new)
* [ES.61: Delete arrays using `delete[]` and non-arrays using `delete`](#Res-del)
* [ES.62: Don't compare pointers into different arrays](#Res-arr2)
* [ES.63: Don't slice](#Res-slice)

Statement rules:

* [ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice](#Res-switch-if)
* [ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice](#Res-for-range)
* [ES.72: Prefer a `for`-statement to a `while`-statement when there is an obvious loop variable](#Res-for-while)
* [ES.73: Prefer a `while`-statement to a `for`-statement when there is no obvious loop variable](#Res-while-for)
* [ES.74: Prefer to declare a loop variable in the initializer part of a `for`-statement](#Res-for-init)
* [ES.75: Avoid `do`-statements](#Res-do)
* [ES.76: Avoid `goto`](#Res-goto)
* [ES.77: ??? `continue`](#Res-continue)
* [ES.78: Always end a non-empty `case` with a `break`](#Res-break)
* [ES.79: ??? `default`](#Res-default)
* [ES.85: Make empty statements visible](#Res-empty)
* [ES.86: Avoid modifying loop control variables inside the body of raw for-loops](#Res-loop-counter)

Arithmetic rules:

* [ES.100: Don't mix signed and unsigned arithmetic](#Res-mix)
* [ES.101: Use unsigned types for bit manipulation](#Res-unsigned)
* [ES.102: Use signed types for arithmetic](#Res-signed)
* [ES.103: Don't overflow](#Res-overflow)
* [ES.104: Don't underflow](#Res-underflow)
* [ES.105: Don't divide by zero](#Res-zero)

### <a name="Res-lib"></a>ES.1: Prefer the standard library to other libraries and to "handcrafted code"

##### Reason

Code using a library can be much easier to write than code working directly with language features, much shorter, tend to be of a higher level of abstraction, and the library code is presumably already tested.
The ISO C++ standard library is among the most widely known and best tested libraries.
It is available as part of all C++ Implementations.

##### Example

    auto sum = accumulate(begin(a), end(a), 0.0);   // good

a range version of `accumulate` would be even better:

    auto sum = accumulate(v, 0.0); // better

but don't hand-code a well-known algorithm:

    int max = v.size();   // bad: verbose, purpose unstated
    double sum = 0.0;
    for (int i = 0; i < max; ++i)
        sum = sum + v[i];

##### Exception

Large parts of the standard library rely on dynamic allocation (free store). These parts, notably the containers but not the algorithms, are unsuitable for some hard-real time and embedded applications. In such cases, consider providing/using similar facilities, e.g.,  a standard-library-style container implemented using a pool allocator.

##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?

### <a name="Res-abstr"></a>ES.2: Prefer suitable abstractions to direct use of language features

##### Reason

A "suitable abstraction" (e.g., library or class) is closer to the application concepts than the bare language, leads to shorter and clearer code, and is likely to be better tested.

##### Example

    vector<string> read1(istream& is)   // good
    {
        vector<string> res;
        for (string s; is >> s;)
            res.push_back(s);
        return res;
    }

The more traditional and lower-level near-equivalent is longer, messier, harder to get right, and most likely slower:

    char** read2(istream& is, int maxelem, int maxstring, int* nread)   // bad: verbose and incomplete
    {
        auto res = new char*[maxelem];
        int elemcount = 0;
        while (is && elemcount < maxelem) {
            auto s = new char[maxstring];
            is.read(s, maxstring);
            res[elemcount++] = s;
        }
        nread = &elemcount;
        return res;
    }

Once the checking for overflow and error handling has been added that code gets quite messy, and there is the problem remembering to `delete` the returned pointer and the C-style strings that array contains.

##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?

## ES.dcl: Declarations

A declaration is a statement. A declaration introduces a name into a scope and may cause the construction of a named object.

### <a name="Res-scope"></a>ES.5: Keep scopes small

##### Reason

Readability. Minimize resource retention. Avoid accidental misuse of value.

**Alternative formulation**: Don't declare a name in an unnecessarily large scope.

##### Example

    void use()
    {
        int i;    // bad: i is needlessly accessible after loop
        for (i = 0; i < 20; ++i) { /* ... */ }
        // no intended use of i here
        for (int i = 0; i < 20; ++i) { /* ... */ }  // good: i is local to for-loop

        if (auto pc = dynamic_cast<Circle*>(ps)) {  // good: pc is local to if-statement
            // ... deal with Circle ...
        }
        else {
            // ... handle error ...
        }
    }

##### Example, bad

    void use(const string& name)
    {
        string fn = name + ".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        // ... 200 lines of code without intended use of fn or is ...
    }

This function is by most measure too long anyway, but the point is that the resources used by `fn` and the file handle held by `is`
are retained for much longer than needed and that unanticipated use of `is` and `fn` could happen later in the function.
In this case, it might be a good idea to factor out the read:

    Record load_record(const string& name)
    {
        string fn = name + ".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        return r;
    }

    void use(const string& name)
    {
        Record r = load_record(name);
        // ... 200 lines of code ...
    }

##### Enforcement

* Flag loop variable declared outside a loop and not used after the loop
* Flag when expensive resources, such as file handles and locks are not used for N-lines (for some suitable N)

### <a name="Res-cond"></a>ES.6: Declare names in for-statement initializers and conditions to limit scope

##### Reason

Readability. Minimize resource retention.

##### Example

    void use()
    {
        for (string s; cin >> s;)
            v.push_back(s);

        for (int i = 0; i < 20; ++i) {   // good: i is local to for-loop
            // ...
        }

        if (auto pc = dynamic_cast<Circle*>(ps)) {   // good: pc is local to if-statement
            // ... deal with Circle ...
        }
        else {
            // ... handle error ...
        }
    }

##### Enforcement

* Flag loop variables declared before the loop and not used after the loop
* (hard) Flag loop variables declared before the loop and used after the loop for an unrelated purpose.

##### C++17 example

Note: C++17 also adds `if` and `switch` initializer statements. These require C++17 support.

    map<int, string> mymap;

    if (auto result = mymap.insert(value); result.second) {
        // insert succeeded, and result is valid for this block
        use(result.first);  // ok
        // ...
    } // result is destroyed here

##### C++17 enforcement (if using a C++17 compiler)

* Flag selection/loop variables declared before the body and not used after the body
* (hard) Flag selection/loop variables declared before the body and used after the body for an unrelated purpose.



### <a name="Res-name-length"></a>ES.7: Keep common and local names short, and keep uncommon and nonlocal names longer

##### Reason

Readability. Lowering the chance of clashes between unrelated non-local names.

##### Example

Conventional short, local names increase readability:

    template<typename T>    // good
    void print(ostream& os, const vector<T>& v)
    {
        for (int i = 0; i < v.size(); ++i)
            os << v[i] << '\n';
    }

An index is conventionally called `i` and there is no hint about the meaning of the vector in this generic function, so `v` is as good name as any. Compare

    template<typename Element_type>   // bad: verbose, hard to read
    void print(ostream& target_stream, const vector<Element_type>& current_vector)
    {
        for (int current_element_index = 0;
                current_element_index < current_vector.size();
                ++current_element_index
        )
        target_stream << current_vector[current_element_index] << '\n';
    }

Yes, it is a caricature, but we have seen worse.

##### Example

Unconventional and short non-local names obscure code:

    void use1(const string& s)
    {
        // ...
        tt(s);   // bad: what is tt()?
        // ...
    }

Better, give non-local entities readable names:

    void use1(const string& s)
    {
        // ...
        trim_tail(s);   // better
        // ...
    }

Here, there is a chance that the reader knows what `trim_tail` means and that the reader can remember it after looking it up.

##### Example, bad

Argument names of large functions are de facto non-local and should be meaningful:

    void complicated_algorithm(vector<Record>& vr, const vector<int>& vi, map<string, int>& out)
    // read from events in vr (marking used Records) for the indices in
    // vi placing (name, index) pairs into out
    {
        // ... 500 lines of code using vr, vi, and out ...
    }

We recommend keeping functions short, but that rule isn't universally adhered to and naming should reflect that.

##### Enforcement

Check length of local and non-local names. Also take function length into account.

### <a name="Res-name-similar"></a>ES.8: Avoid similar-looking names

##### Reason

Code clarity and readability. Too-similar names slow down comprehension and increase the likelihood of error.

##### Example; bad

    if (readable(i1 + l1 + ol + o1 + o0 + ol + o1 + I0 + l0)) surprise();

##### Example; bad

Do not declare a non-type with the same name as a type in the same scope. This removes the need to disambiguate with a keyword such as `struct` or `enum`. It also removes a source of errors, as `struct X` can implicitly declare `X` if lookup fails.

    struct foo { int n; };
    struct foo foo();       // BAD, foo is a type already in scope
    struct foo x = foo();   // requires disambiguation

##### Exception

Antique header files might declare non-types and types with the same name in the same scope.

##### Enforcement

* Check names against a list of known confusing letter and digit combinations.
* Flag a declaration of a variable, function, or enumerator that hides a class or enumeration declared in the same scope.

### <a name="Res-not-CAPS"></a>ES.9: Avoid `ALL_CAPS` names

##### Reason

Such names are commonly used for macros. Thus, `ALL_CAPS` name are vulnerable to unintended macro substitution.

##### Example

    // somewhere in some header:
    #define NE !=

    // somewhere else in some other header:
    enum Coord { N, NE, NW, S, SE, SW, E, W };

    // somewhere third in some poor programmer's .cpp:
    switch (direction) {
    case N:
        // ...
    case NE:
        // ...
    // ...
    }

##### Note

Do not use `ALL_CAPS` for constants just because constants used to be macros.

##### Enforcement

Flag all uses of ALL CAPS. For older code, accept ALL CAPS for macro names and flag all non-ALL-CAPS macro names.

### <a name="Res-name-one"></a>ES.10: Declare one name (only) per declaration

##### Reason

One-declaration-per line increases readability and avoids mistakes related to
the C/C++ grammar. It also leaves room for a more descriptive end-of-line
comment.

##### Example, bad

    char *p, c, a[7], *pp[7], **aa[10];   // yuck!

##### Exception

A function declaration can contain several function argument declarations.

##### Example

    template <class InputIterator, class Predicate>
    bool any_of(InputIterator first, InputIterator last, Predicate pred);

or better using concepts:

    bool any_of(InputIterator first, InputIterator last, Predicate pred);

##### Example

    double scalbn(double x, int n);   // OK: x * pow(FLT_RADIX, n); FLT_RADIX is usually 2

or:

    double scalbn(    // better: x * pow(FLT_RADIX, n); FLT_RADIX is usually 2
        double x,     // base value
        int n         // exponent
    );

or:

    // better: base * pow(FLT_RADIX, exponent); FLT_RADIX is usually 2
    double scalbn(double base, int exponent);

##### Example

    int a=7, b=9, c, d=10, e=3;

In a long list of declarators is is easy to overlook an uninitializeed variable.

##### Enforcement

Flag variable and constant declarations with multiple declarators (e.g., `int* p, q;`)

### <a name="Res-auto"></a>ES.11: Use `auto` to avoid redundant repetition of type names

##### Reason

* Simple repetition is tedious and error prone.
* When you use `auto`, the name of the declared entity is in a fixed position in the declaration, increasing readability.
* In a template function declaration the return type can be a member type.

##### Example

Consider:

    auto p = v.begin();   // vector<int>::iterator
    auto s = v.size();
    auto h = t.future();
    auto q = make_unique<int[]>(s);
    auto f = [](int x){ return x + 10; };

In each case, we save writing a longish, hard-to-remember type that the compiler already knows but a programmer could get wrong.

##### Example

    template<class T>
    auto Container<T>::first() -> Iterator;   // Container<T>::Iterator

##### Exception

Avoid `auto` for initializer lists and in cases where you know exactly which type you want and where an initializer might require conversion.

##### Example

    auto lst = { 1, 2, 3 };   // lst is an initializer list
    auto x{1};   // x is an int (after correction of the C++14 standard; initializer_list in C++11)

##### Note

When concepts become available, we can (and should) be more specific about the type we are deducing:

    // ...
    ForwardIterator p = algo(x, y, z);

##### Enforcement

Flag redundant repetition of type names in a declaration.

### <a name="Res-reuse"></a>ES.12: Do not reuse names in nested scopes

##### Reason

It is easy to get confused about which variable is used.
Can cause maintenance problems.

##### Example, bad

    int d = 0;
    // ...
    if (cond) {
        // ...
        d = 9;
        // ...
    }
    else {
        // ...
        int d = 7;
        // ...
        d = value_to_be_returned;
        // ...
    }

    return d;

If this is a large `if`-statement, it is easy to overlook that a new `d` has been introduced in the inner scope.
This is a known source of bugs.
Sometimes such reuse of a name in an inner scope is called "shadowing".

##### Note

Shadowing is primarily a problem when functions are too large and too complex.

##### Example

Shadowing of function arguments in the outermost block is disallowed by the language:

    void f(int x)
    {
        int x = 4;  // error: reuse of function argument name

        if (x) {
            int x = 7;  // allowed, but bad
            // ...
        }
    }

##### Example, bad

Reuse of a member name as a local variable can also be a problem:

    struct S {
        int m;
        void f(int x);
    };

    void S::f(int x)
    {
        m = 7;    // assign to member
        if (x) {
            int m = 9;
            // ...
            m = 99; // assign to member
            // ...
        }
    }

##### Exception

We often reuse function names from a base class in a derived class:

    struct B {
        void f(int);
    };

    struct D : B {
        void f(double);
        using B::f;
    };

This is error-prone.
For example, had we forgotten the using declaration, a call `d.f(1)` would not have found the `int` version of `f`.

??? Do we need a specific rule about shadowing/hiding in class hierarchies?

##### Enforcement

* Flag reuse of a name in nested local scopes
* Flag reuse of a member name as a local variable in a member function
* Flag reuse of a global name as a local variable or a member name
* Flag reuse of a base class member name in a derived class (except for function names)

### <a name="Res-always"></a>ES.20: Always initialize an object

##### Reason

Avoid used-before-set errors and their associated undefined behavior.
Avoid problems with comprehension of complex initialization.
Simplify refactoring.

##### Example

    void use(int arg)
    {
        int i;   // bad: uninitialized variable
        // ...
        i = 7;   // initialize i
    }

No, `i = 7` does not initialize `i`; it assigns to it. Also, `i` can be read in the `...` part. Better:

    void use(int arg)   // OK
    {
        int i = 7;   // OK: initialized
        string s;    // OK: default initialized
        // ...
    }

##### Note

The *always initialize* rule is deliberately stronger than the *an object must be set before used* language rule.
The latter, more relaxed rule, catches the technical bugs, but:

* It leads to less readable code
* It encourages people to declare names in greater than necessary scopes
* It leads to harder to read code
* It leads to logic bugs by encouraging complex code
* It hampers refactoring

The *always initialize* rule is a style rule aimed to improve maintainability as well as a rule protecting against used-before-set errors.

##### Example

Here is an example that is often considered to demonstrate the need for a more relaxed rule for initialization

    widget i;    // "widget" a type that's expensive to initialize, possibly a large POD
    widget j;

    if (cond) {  // bad: i and j are initialized "late"
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }

This cannot trivially be rewritten to initialize `i` and `j` with initializers.
Note that for types with a default constructor, attempting to postpone initialization simply leads to a default initialization followed by an assignment.
A popular reason for such examples is "efficiency", but a compiler that can detect whether we made a used-before-set error can also eliminate any redundant double initialization.

At the cost of repeating `cond` we could write:

    widget i = (cond) ? f1() : f3();
    widget j = (cond) ? f2() : f4();

Assuming that there is a logical connection between `i` and `j`, that connection should probably be expressed in code:

    pair<widget, widget> make_related_widgets(bool x)
    {
        return (x) ? {f1(), f2()} : {f3(), f4() };
    }

    auto init = make_related_widgets(cond);
    widget i = init.first;
    widget j = init.second;

Obviously, what we really would like is a construct that initialized n variables from a `tuple`. For example:

    auto [i, j] = make_related_widgets(cond);    // C++17, not C++14

Today, we might approximate that using `tie()`:

    widget i;       // bad: uninitialized variable
    widget j;
    tie(i, j) = make_related_widgets(cond);

This may be seen as an example of the *immediately initialize from input* exception below.

Creating optimal and equivalent code from all of these examples should be well within the capabilities of modern C++ compilers
(but don't make performance claims without measuring; a compiler may very well not generate optimal code for every example and
there may be language rules preventing some optimization that you would have liked in a particular case).

##### Note

Complex initialization has been popular with clever programmers for decades.
It has also been a major source of errors and complexity.
Many such errors are introduced during maintenance years after the initial implementation.

##### Exception

It you are declaring an object that is just about to be initialized from input, initializing it would cause a double initialization.
However, beware that this may leave uninitialized data beyond the input -- and that has been a fertile source of errors and security breaches:

    constexpr int max = 8 * 1024;
    int buf[max];         // OK, but suspicious: uninitialized
    f.read(buf, max);

The cost of initializing that array could be significant in some situations.
However, such examples do tend to leave uninitialized variables accessible, so they should be treated with suspicion.

    constexpr int max = 8 * 1024;
    int buf[max] = {};   // zero all elements; better in some situations
    f.read(buf, max);

When feasible use a library function that is known not to overflow. For example:

    string s;   // s is default initialized to ""
    cin >> s;   // s expands to hold the string

Don't consider simple variables that are targets for input operations exceptions to this rule:

    int i;   // bad
    // ...
    cin >> i;

In the not uncommon case where the input target and the input operation get separated (as they should not) the possibility of used-before-set opens up.

    int i2 = 0;   // better
    // ...
    cin >> i;

A good optimizer should know about input operations and eliminate the redundant operation.

##### Example

Using an `uninitialized` or sentinel value is a symptom of a problem and not a
solution:

    widget i = uninit;  // bad
    widget j = uninit;

    // ...
    use(i);         // possibly used before set
    // ...

    if (cond) {     // bad: i and j are initialized "late"
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }

Now the compiler cannot even simply detect a used-before-set. Further, we've introduced complexity in the state space for widget: which operations are valid on an `uninit` widget and which are not?

##### Note

Sometimes, a lambda can be used as an initializer to avoid an uninitialized variable:

    error_code ec;
    Value v = [&] {
        auto p = get_value();   // get_value() returns a pair<error_code, Value>
        ec = p.first;
        return p.second;
    }();

or maybe:

    Value v = [] {
        auto p = get_value();   // get_value() returns a pair<error_code, Value>
        if (p.first) throw Bad_value{p.first};
        return p.second;
    }();

**See also**: [ES.28](#Res-lambda-init)

##### Enforcement

* Flag every uninitialized variable.
  Don't flag variables of user-defined types with default constructors.
* Check that an uninitialized buffer is written into *immediately* after declaration.
  Passing an uninitialized variable as a reference to non-`const` argument can be assumed to be a write into the variable.

### <a name="Res-introduce"></a>ES.21: Don't introduce a variable (or constant) before you need to use it

##### Reason

Readability. To limit the scope in which the variable can be used.

##### Example

    int x = 7;
    // ... no use of x here ...
    ++x;

##### Enforcement

Flag declarations that are distant from their first use.

### <a name="Res-init"></a>ES.22: Don't declare a variable until you have a value to initialize it with

##### Reason

Readability. Limit the scope in which a variable can be used. Don't risk used-before-set. Initialization is often more efficient than assignment.

##### Example, bad

    string s;
    // ... no use of s here ...
    s = "what a waste";

##### Example, bad

    SomeLargeType var;   // ugly CaMeLcAsEvArIaBlE

    if (cond)   // some non-trivial condition
        Set(&var);
    else if (cond2 || !cond3) {
        var = Set2(3.14);
    }
    else {
        var = 0;
        for (auto& e : something)
            var += e;
    }

    // use var; that this isn't done too early can be enforced statically with only control flow

This would be fine if there was a default initialization for `SomeLargeType` that wasn't too expensive.
Otherwise, a programmer might very well wonder if every possible path through the maze of conditions has been covered.
If not, we have a "use before set" bug. This is a maintenance trap.

For initializers of moderate complexity, including for `const` variables, consider using a lambda to express the initializer; see [ES.28](#Res-lambda-init).

##### Enforcement

* Flag declarations with default initialization that are assigned to before they are first read.
* Flag any complicated computation after an uninitialized variable and before its use.

### <a name="Res-list"></a>ES.23: Prefer the `{}` initializer syntax

##### Reason

The rules for `{}` initialization are simpler, more general, less ambiguous, and safer than for other forms of initialization.

##### Example

    int x {f(99)};
    vector<int> v = {1, 2, 3, 4, 5, 6};

##### Exception

For containers, there is a tradition for using `{...}` for a list of elements and `(...)` for sizes:

    vector<int> v1(10);    // vector of 10 elements with the default value 0
    vector<int> v2 {10};   // vector of 1 element with the value 10

##### Note

`{}`-initializers do not allow narrowing conversions.

##### Example

    int x {7.9};   // error: narrowing
    int y = 7.9;   // OK: y becomes 7. Hope for a compiler warning

##### Note

`{}` initialization can be used for all initialization; other forms of initialization can't:

    auto p = new vector<int> {1, 2, 3, 4, 5};   // initialized vector
    D::D(int a, int b) :m{a, b} {   // member initializer (e.g., m might be a pair)
        // ...
    };
    X var {};   // initialize var to be empty
    struct S {
        int m {7};   // default initializer for a member
        // ...
    };

##### Note

Initialization of a variable declared using `auto` with a single value, e.g., `{v}`, had surprising results until recently:

    auto x1 {7};        // x1 is an int with the value 7
    // x2 is an initializer_list<int> with an element 7
    // (this will will change to "element 7" in C++17)
    auto x2 = {7};

    auto x11 {7, 8};    // error: two initializers
    auto x22 = {7, 8};  // x2 is an initializer_list<int> with elements 7 and 8

##### Exception

Use `={...}` if you really want an `initializer_list<T>`

    auto fib10 = {0, 1, 2, 3, 5, 8, 13, 21, 34, 55};   // fib10 is a list

##### Note

Old habits die hard, so this rule is hard to apply consistently, especially as there are so many cases where `=` is innocent.

##### Example

    template<typename T>
    void f()
    {
        T x1(1);    // T initialized with 1
        T x0();     // bad: function declaration (often a mistake)

        T y1 {1};   // T initialized with 1
        T y0 {};    // default initialized T
        // ...
    }

**See also**: [Discussion](#???)

##### Enforcement

Tricky.

* Don't flag uses of `=` for simple initializers.
* Look for `=` after `auto` has been seen.

### <a name="Res-unique"></a>ES.24: Use a `unique_ptr<T>` to hold pointers

##### Reason

Using `std::unique_ptr` is the simplest way to avoid leaks. It is reliable, it
makes the type system do much of the work to validate ownership safety, it
increases readability, and it has zero or near zero runtime cost.

##### Example

    void use(bool leak)
    {
        auto p1 = make_unique<int>(7);   // OK
        int* p2 = new int{7};            // bad: might leak
        // ...
        if (leak) return;
        // ...
    }

If `leak == true` the object pointed to by `p2` is leaked and the object pointed to by `p1` is not.

##### Enforcement

Look for raw pointers that are targets of `new`, `malloc()`, or functions that may return such pointers.

### <a name="Res-const"></a>ES.25: Declare an object `const` or `constexpr` unless you want to modify its value later on

##### Reason

That way you can't change the value by mistake. That way may offer the compiler optimization opportunities.

##### Example

    void f(int n)
    {
        const int bufmax = 2 * n + 2;  // good: we can't change bufmax by accident
        int xmax = n;                  // suspicious: is xmax intended to change?
        // ...
    }

##### Enforcement

Look to see if a variable is actually mutated, and flag it if
not. Unfortunately, it may be impossible to detect when a non-`const` was not
*intended* to vary (vs when it merely did not vary).

### <a name="Res-recycle"></a>ES.26: Don't use a variable for two unrelated purposes

##### Reason

Readability and safety.

##### Example, bad

    void use()
    {
        int i;
        for (i = 0; i < 20; ++i) { /* ... */ }
        for (i = 0; i < 200; ++i) { /* ... */ } // bad: i recycled
    }

##### Note

As an optimization, you may want to reuse a buffer as a scratch pad, but even then prefer to limit the variable's scope as much as possible and be careful not to cause bugs from data left in a recycled buffer as this is a common source of security bugs.

    {
        std::string buffer;             // to avoid reallocations on every loop iteration
        for (auto& o : objects)
        {
            // First part of the work.
            generateFirstString(buffer, o);
            writeToFile(buffer);

            // Second part of the work.
            generateSecondString(buffer, o);
            writeToFile(buffer);

            // etc...
        }
    }

##### Enforcement

Flag recycled variables.

### <a name="Res-stack"></a>ES.27: Use `std::array` or `stack_array` for arrays on the stack

##### Reason

They are readable and don't implicitly convert to pointers.
They are not confused with non-standard extensions of built-in arrays.

##### Example, bad

    const int n = 7;
    int m = 9;

    void f()
    {
        int a1[n];
        int a2[m];   // error: not ISO C++
        // ...
    }

##### Note

The definition of `a1` is legal C++ and has always been.
There is a lot of such code.
It is error-prone, though, especially when the bound is non-local.
Also, it is a "popular" source of errors (buffer overflow, pointers from array decay, etc.).
The definition of `a2` is C but not C++ and is considered a security risk

##### Example

    const int n = 7;
    int m = 9;

    void f()
    {
        array<int, n> a1;
        stack_array<int> a2(m);
        // ...
    }

##### Enforcement

* Flag arrays with non-constant bounds (C-style VLAs)
* Flag arrays with non-local constant bounds

### <a name="Res-lambda-init"></a>ES.28: Use lambdas for complex initialization, especially of `const` variables

##### Reason

It nicely encapsulates local initialization, including cleaning up scratch variables needed only for the initialization, without needing to create a needless nonlocal yet nonreusable function. It also works for variables that should be `const` but only after some initialization work.

##### Example, bad

    widget x;   // should be const, but:
    for (auto i = 2; i <= N; ++i) {             // this could be some
        x += some_obj.do_something_with(i);  // arbitrarily long code
    }                                        // needed to initialize x
    // from here, x should be const, but we can't say so in code in this style

##### Example, good

    const widget x = [&]{
        widget val;                                // assume that widget has a default constructor
        for (auto i = 2; i <= N; ++i) {            // this could be some
            val += some_obj.do_something_with(i);  // arbitrarily long code
        }                                          // needed to initialize x
        return val;
    }();

##### Example

    string var = [&]{
        if (!in) return "";   // default
        string s;
        for (char c : in >> c)
            s += toupper(c);
        return s;
    }(); // note ()

If at all possible, reduce the conditions to a simple set of alternatives (e.g., an `enum`) and don't mix up selection and initialization.

##### Example

    owner<istream&> in = [&]{
        switch (source) {
        case default:       owned = false; return cin;
        case command_line:  owned = true;  return *new istringstream{argv[2]};
        case file:          owned = true;  return *new ifstream{argv[2]};
    }();

##### Enforcement

Hard. At best a heuristic. Look for an uninitialized variable followed by a loop assigning to it.

### <a name="Res-macros"></a>ES.30: Don't use macros for program text manipulation

##### Reason

Macros are a major source of bugs.
Macros don't obey the usual scope and type rules.
Macros ensure that the human reader sees something different from what the compiler sees.
Macros complicate tool building.

##### Example, bad

    #define Case break; case   /* BAD */

This innocuous-looking macro makes a single lower case `c` instead of a `C` into a bad flow-control bug.

##### Note

This rule does not ban the use of macros for "configuration control" use in `#ifdef`s, etc.

##### Enforcement

Scream when you see a macro that isn't just used for source control (e.g., `#ifdef`)

### <a name="Res-macros2"></a>ES.31: Don't use macros for constants or "functions"

##### Reason

Macros are a major source of bugs.
Macros don't obey the usual scope and type rules.
Macros don't obey the usual rules for argument passing.
Macros ensure that the human reader sees something different from what the compiler sees.
Macros complicate tool building.

##### Example, bad

    #define PI 3.14
    #define SQUARE(a, b) (a * b)

Even if we hadn't left a well-known bug in `SQUARE` there are much better behaved alternatives; for example:

    constexpr double pi = 3.14;
    template<typename T> T square(T a, T b) { return a * b; }

##### Enforcement

Scream when you see a macro that isn't just used for source control (e.g., `#ifdef`)

### <a name="Res-ALL_CAPS"></a>ES.32: Use `ALL_CAPS` for all macro names

##### Reason

Convention. Readability. Distinguishing macros.

##### Example

    #define forever for (;;)   /* very BAD */

    #define FOREVER for (;;)   /* Still evil, but at least visible to humans */

##### Enforcement

Scream when you see a lower case macro.

### <a name="Res-MACROS"></a>ES.33: If you must use macros, give them unique names

##### Reason

Macros do not obey scope rules.

##### Example

    #define MYCHAR        /* BAD, will eventually clash with someone else's MYCHAR*/

    #define ZCORP_CHAR    /* Still evil, but less likely to clash */

##### Note

Avoid macros if you can: [ES.30](#Res-macros), [ES.31](#Res-macros2), and [ES.32](#Res-ALL_CAPS).
However, there are billions of lines of code littered with macros and a long tradition for using and overusing macros.
If you are forced to use macros, use long names and supposedly unique prefixes (e.g., your organization's name) to lower the likelihood of a clash.

##### Enforcement

Warn against short macro names.

### <a name="Res-ellipses"></a> ES.34: Don't define a (C-style) variadic function

##### Reason

Not type safe.
Requires messy cast-and-macro-laden code to get working right.

##### Example

    #include <cstdarg>

    // "severity" followed by a zero-terminated list of char*s; write the C-style strings to cerr
    void error(int severity ...)
    {
        va_list ap;             // a magic type for holding arguments
        va_start(ap, severity); // arg startup: "severity" is the first argument of error()

        for (;;) {
            // treat the next var as a char*; no checking: a cast in disguise
            char* p = va_arg(ap, char*);
            if (p == nullptr) break;
            cerr << p << ' ';
        }

        va_end(ap);             // arg cleanup (don't forget this)

        cerr << '\n';
        if (severity) exit(severity);
    }

    void use()
    {
        error(7, "this", "is", "an", "error", nullptr);
        error(7); // crash
        error(7, "this", "is", "an", "error");  // crash
        const char* is = "is";
        string an = "an";
        error(7, "this", "is", an, "error"); // crash
    }

**Alternative**: Overloading. Templates. Variadic templates.

##### Note

This is basically the way `printf` is implemented.

##### Enforcement

* Flag definitions of C-style variadic functions.
* Flag `#include <cstdarg>` and `#include <stdarg.h>`

## ES.stmt: Statements

Statements control the flow of control (except for function calls and exception throws, which are expressions).

### <a name="Res-switch-if"></a>ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice

##### Reason

* Readability.
* Efficiency: A `switch` compares against constants and is usually better optimized than a series of tests in an `if`-`then`-`else` chain.
* A `switch` enables some heuristic consistency checking. For example, have all values of an `enum` been covered? If not, is there a `default`?

##### Example

    void use(int n)
    {
        switch (n) {   // good
        case 0:   // ...
        case 7:   // ...
        }
    }

rather than:

    void use2(int n)
    {
        if (n == 0)   // bad: if-then-else chain comparing against a set of constants
            // ...
        else if (n == 7)
            // ...
    }

##### Enforcement

Flag `if`-`then`-`else` chains that check against constants (only).

### <a name="Res-for-range"></a>ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice

##### Reason

Readability. Error prevention. Efficiency.

##### Example

    for (int i = 0; i < v.size(); ++i)   // bad
            cout << v[i] << '\n';

    for (auto p = v.begin(); p != v.end(); ++p)   // bad
        cout << *p << '\n';

    for (auto& x : v)    // OK
        cout << x << '\n';

    for (int i = 1; i < v.size(); ++i) // touches two elements: can't be a range-for
        cout << v[i] + v[i - 1] << '\n';

    for (int i = 0; i < v.size(); ++i) // possible side-effect: can't be a range-for
        cout << f(v, &v[i]) << '\n';

    for (int i = 0; i < v.size(); ++i) { // body messes with loop variable: can't be a range-for
        if (i % 2 == 0)
            continue;   // skip even elements
        else
            cout << v[i] << '\n';
    }

A human or a good static analyzer may determine that there really isn't a side effect on `v` in `f(v, &v[i])` so that the loop can be rewritten.

"Messing with the loop variable" in the body of a loop is typically best avoided.

##### Note

Don't use expensive copies of the loop variable of a range-`for` loop:

    for (string s : vs) // ...

This will copy each elements of `vs` into `s`. Better:

    for (string& s : vs) // ...

Better still, if the loop variable isn't modified or copied:

    for (const string& s : vs) // ...

##### Enforcement

Look at loops, if a traditional loop just looks at each element of a sequence, and there are no side-effects on what it does with the elements, rewrite the loop to a ranged-`for` loop.

### <a name="Res-for-while"></a>ES.72: Prefer a `for`-statement to a `while`-statement when there is an obvious loop variable

##### Reason

Readability: the complete logic of the loop is visible "up front". The scope of the loop variable can be limited.

##### Example

    for (int i = 0; i < vec.size(); i++) {
        // do work
    }

##### Example, bad

    int i = 0;
    while (i < vec.size()) {
        // do work
        i++;
    }

##### Enforcement

???

### <a name="Res-while-for"></a>ES.73: Prefer a `while`-statement to a `for`-statement when there is no obvious loop variable

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-for-init"></a>ES.74: Prefer to declare a loop variable in the initializer part of a `for`-statement

##### Reason

Limit the loop variable visibility to the scope of the loop.
Avoid using the loop variable for other purposes after the loop.

##### Example

    for (int i = 0; i < 100; ++i) {   // GOOD: i var is visible only inside the loop
        // ...
    }

##### Example, don't

    int j;                            // BAD: j is visible outside the loop
    for (j = 0; j < 100; ++j) {
        // ...
    }
    // j is still visible here and isn't needed

**See also**: [Don't use a variable for two unrelated purposes](#Res-recycle)

##### Enforcement

Warn when a variable modified inside the `for`-statement is declared outside the loop and not being used outside the loop.

**Discussion**: Scoping the loop variable to the loop body also helps code optimizers greatly. Recognizing that the induction variable
is only accessible in the loop body unblocks optimizations such as hoisting, strength reduction, loop-invariant code motion, etc.

### <a name="Res-do"></a>ES.75: Avoid `do`-statements

##### Reason

Readability, avoidance of errors.
The termination condition is at the end (where it can be overlooked) and the condition is not checked the first time through. ???

##### Example

    int x;
    do {
        cin >> x;
        // ...
    } while (x < 0);

##### Enforcement

???

### <a name="Res-goto"></a>ES.76: Avoid `goto`

##### Reason

Readability, avoidance of errors. There are better control structures for humans; `goto` is for machine generated code.

##### Exception

Breaking out of a nested loop. In that case, always jump forwards.

##### Example

    ???

##### Example

There is a fair amount of use of the C goto-exit idiom:

    void f()
    {
        // ...
            goto exit;
        // ...
            goto exit;
        // ...
    exit:
        ... common cleanup code ...
    }

This is an ad-hoc simulation of destructors. Declare your resources with handles with destructors that clean up.

##### Enforcement

* Flag `goto`. Better still flag all `goto`s that do not jump from a nested loop to the statement immediately after a nest of loops.

### <a name="Res-continue"></a>ES.77: ??? `continue`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-break"></a>ES.78: Always end a non-empty `case` with a `break`

##### Reason

 Accidentally leaving out a `break` is a fairly common bug.
 A deliberate fallthrough is a maintenance hazard.

##### Example

    switch (eventType)
    {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
    case Error:
        display_error_window(); // Bad
        break;
    }

It is easy to overlook the fallthrough. Be explicit:

    switch (eventType)
    {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
        // fallthrough
    case Error:
        display_error_window(); // Bad
        break;
    }

There is a proposal for a `[[fallthrough]]` annotation.

##### Note

Multiple case labels of a single statement is OK:

    switch (x) {
    case 'a':
    case 'b':
    case 'f':
        do_something(x);
        break;
    }

##### Enforcement

Flag all fallthroughs from non-empty `case`s.

### <a name="Res-default"></a>ES.79: ??? `default`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-empty"></a>ES.85: Make empty statements visible

##### Reason

Readability.

##### Example

    for (i = 0; i < max; ++i);   // BAD: the empty statement is easily overlooked
    v[i] = f(v[i]);

    for (auto x : v) {           // better
        // nothing
    }
    v[i] = f(v[i]);

##### Enforcement

Flag empty statements that are not blocks and don't contain comments.

### <a name="Res-loop-counter"></a>ES.86: Avoid modifying loop control variables inside the body of raw for-loops

##### Reason

The loop control up front should enable correct reasoning about what is happening inside the loop. Modifying loop counters in both the iteration-expression and inside the body of the loop is a perennial source of surprises and bugs.

##### Example

    for (int i = 0; i < 10; ++i) {
        // no updates to i -- ok
    }

    for (int i = 0; i < 10; ++i) {
        //
        if (/* something */) ++i; // BAD
        //
    }

    bool skip = false;
    for (int i = 0; i < 10; ++i) {
        if (skip) { skip = false; continue; }
        //
        if (/* something */) skip = true;  // Better: using two variable for two concepts.
        //
    }

##### Enforcement

Flag variables that are potentially updated (have a non-const use) in both the loop control iteration-expression and the loop body.

## ES.expr: Expressions

Expressions manipulate values.

### <a name="Res-complicated"></a>ES.40: Avoid complicated expressions

##### Reason

Complicated expressions are error-prone.

##### Example

    // bad: assignment hidden in subexpression
    while ((c = getc()) != -1)

    // bad: two non-local variables assigned in a sub-expressions
    while ((cin >> c1, cin >> c2), c1 == c2)

    // better, but possibly still too complicated
    for (char c1, c2; cin >> c1 >> c2 && c1 == c2;)

    // OK: if i and j are not aliased
    int x = ++i + ++j;

    // OK: if i != j and i != k
    v[i] = v[j] + v[k];

    // bad: multiple assignments "hidden" in subexpressions
    x = a + (b = f()) + (c = g()) * 7;

    // bad: relies on commonly misunderstood precedence rules
    x = a & b + c * d && e ^ f == 7;

    // bad: undefined behavior
    x = x++ + x++ + ++x;

Some of these expressions are unconditionally bad (e.g., they rely on undefined behavior). Others are simply so complicated and/or unusual that even good programmers could misunderstand them or overlook a problem when in a hurry.

##### Note

A programmer should know and use the basic rules for expressions.

##### Example

    x = k * y + z;             // OK

    auto t1 = k * y;           // bad: unnecessarily verbose
    x = t1 + z;

    if (0 <= x && x < max)   // OK

    auto t1 = 0 <= x;        // bad: unnecessarily verbose
    auto t2 = x < max;
    if (t1 && t2)            // ...

##### Enforcement

Tricky. How complicated must an expression be to be considered complicated? Writing computations as statements with one operation each is also confusing. Things to consider:

* side effects: side effects on multiple non-local variables (for some definition of non-local) can be suspect, especially if the side effects are in separate subexpressions
* writes to aliased variables
* more than N operators (and what should N be?)
* reliance of subtle precedence rules
* uses undefined behavior (can we catch all undefined behavior?)
* implementation defined behavior?
* ???

### <a name="Res-parens"></a>ES.41: If in doubt about operator precedence, parenthesize

##### Reason

Avoid errors. Readability. Not everyone has the operator table memorized.

##### Example

    const unsigned int flag = 2;
    unsigned int a = flag;

    if (a & flag != 0)  // bad: means a&(flag != 0)

Note: We recommend that programmers know their precedence table for the arithmetic operations, the logical operations, but consider mixing bitwise logical operations with other operators in need of parentheses.

    if ((a & flag) != 0)  // OK: works as intended

##### Note

You should know enough not to need parentheses for:

    if (a < 0 || a <= max) {
        // ...
    }

##### Enforcement

* Flag combinations of bitwise-logical operators and other operators.
* Flag assignment operators not as the leftmost operator.
* ???

### <a name="Res-ptr"></a>ES.42: Keep use of pointers simple and straightforward

##### Reason

Complicated pointer manipulation is a major source of errors.

* Do all pointer arithmetic on a `span` (exception ++p in simple loop???)
* Avoid pointers to pointers
* ???

##### Example

    ???

##### Enforcement

We need a heuristic limiting the complexity of pointer arithmetic statement.

### <a name="Res-order"></a>ES.43: Avoid expressions with undefined order of evaluation

##### Reason

You have no idea what such code does. Portability.
Even if it does something sensible for you, it may do something different on another compiler (e.g., the next release of your compiler) or with a different optimizer setting.

##### Example

    v[i] = ++i;   //  the result is undefined

A good rule of thumb is that you should not read a value twice in an expression where you write to it.

##### Example

    ???

##### Note

What is safe?

##### Enforcement

Can be detected by a good analyzer.

### <a name="Res-order-fct"></a>ES.44: Don't depend on order of evaluation of function arguments

##### Reason

Because that order is unspecified.

##### Example

    int i = 0;
    f(++i, ++i);

The call will most likely be `f(0, 1)` or `f(1, 0)`, but you don't know which. Technically, the behavior is undefined.

##### Example

??? overloaded operators can lead to order of evaluation problems (shouldn't :-()

    f1()->m(f2());   // m(f1(), f2())
    cout << f1() << f2();   // operator<<(operator<<(cout, f1()), f2())

##### Enforcement

Can be detected by a good analyzer.

### <a name="Res-magic"></a>ES.45: Avoid "magic constants"; use symbolic constants

##### Reason

Unnamed constants embedded in expressions are easily overlooked and often hard to understand:

##### Example

    for (int m = 1; m <= 12; ++m)   // don't: magic constant 12
        cout << month[m] << '\n';

No, we don't all know that there are 12 months, numbered 1..12, in a year. Better:

    constexpr int month_count = 12;   // months are numbered 1..12

    for (int m = first_month; m <= month_count; ++m)   // better
        cout << month[m] << '\n';

Better still, don't expose constants:

    for (auto m : month)
        cout << m << '\n';

##### Enforcement

Flag literals in code. Give a pass to `0`, `1`, `nullptr`, `\n`, `""`, and others on a positive list.

### <a name="Res-narrowing"></a>ES.46: Avoid lossy (narrowing, truncating) arithmetic conversions

##### Reason

A narrowing conversion destroys information, often unexpectedly so.

##### Example, bad

A key example is basic narrowing:

    double d = 7.9;
    int i = d;    // bad: narrowing: i becomes 7
    i = (int) d;  // bad: we're going to claim this is still not explicit enough

    void f(int x, long y, double d)
    {
        char c1 = x;   // bad: narrowing
        char c2 = y;   // bad: narrowing
        char c3 = d;   // bad: narrowing
    }

##### Note

The guideline support library offers a `narrow` operation for specifying that narrowing is acceptable and a `narrow` ("narrow if") that throws an exception if a narrowing would throw away information:

    i = narrow_cast<int>(d);   // OK (you asked for it): narrowing: i becomes 7
    i = narrow<int>(d);        // OK: throws narrowing_error

We also include lossy arithmetic casts, such as from a negative floating point type to an unsigned integral type:

    double d = -7.9;
    unsigned u = 0;

    u = d;                          // BAD
    u = narrow_cast<unsigned>(d);   // OK (you asked for it): u becomes 0
    u = narrow<unsigned>(d);        // OK: throws narrowing_error

##### Enforcement

A good analyzer can detect all narrowing conversions. However, flagging all narrowing conversions will lead to a lot of false positives. Suggestions:

* flag all floating-point to integer conversions (maybe only `float`->`char` and `double`->`int`. Here be dragons! we need data)
* flag all `long`->`char` (I suspect `int`->`char` is very common. Here be dragons! we need data)
* consider narrowing conversions for function arguments especially suspect

### <a name="Res-nullptr"></a>ES.47: Use `nullptr` rather than `0` or `NULL`

##### Reason

Readability. Minimize surprises: `nullptr` cannot be confused with an
`int`. `nullptr` also has a well-specified (very restrictive) type, and thus
works in more scenarios where type deduction might do the wrong thing on `NULL`
or `0`.

##### Example

Consider:

    void f(int);
    void f(char*);
    f(0);         // call f(int)
    f(nullptr);   // call f(char*)

##### Enforcement

Flag uses of `0` and `NULL` for pointers. The transformation may be helped by simple program transformation.

### <a name="Res-casts"></a>ES.48: Avoid casts

##### Reason

Casts are a well-known source of errors. Makes some optimizations unreliable.

##### Example

    ???

##### Note

Programmer who write casts typically assumes that they know what they are doing.
In fact, they often disable the general rules for using values.
Overload resolution and template instantiation usually pick the right function if there is a right function to pick.
If there is not, maybe there ought to be, rather than applying a local fix (cast).

##### Note

Casts are necessary in a systems programming language.  For example, how else
would we get the address of a device register into a pointer?  However, casts
are seriously overused as well as a major source of errors.

##### Note

If you feel the need for a lot of casts, there may be a fundamental design problem.

##### Enforcement

* Force the elimination of C-style casts
* Warn against named casts
* Warn if there are many functional style casts (there is an obvious problem in quantifying 'many').

### <a name="Res-casts-named"></a>ES.49: If you must use a cast, use a named cast

##### Reason

Readability. Error avoidance.
Named casts are more specific than a C-style or functional cast, allowing the compiler to catch some errors.

The named casts are:

* `static_cast`
* `const_cast`
* `reinterpret_cast`
* `dynamic_cast`
* `std::move`         // `move(x)` is an rvalue reference to `x`
* `std::forward`      // `forward(x)` is an rvalue reference to `x`
* `gsl::narrow_cast`  // `narrow_cast<T>(x)` is `static_cast<T>(x)`
* `gsl::narrow`       // `narrow<T>(x)` is `static_cast<T>(x)` if `static_cast<T>(x) == x` or it throws `narrowing_error`

##### Example

    ???

##### Note

When converting between types with no information loss (e.g. from `float` to
`double` or `int64` from `int32`), brace initialization may be used instead.

    double d{some_float};
    int64_t i{some_int32};

This makes it clear that the type conversion was intended and also prevents
conversions between types that might result in loss of precision. (It is a
compilation error to try to initialize a `float` from a `double` in this fashion,
for example.)

##### Enforcement

Flag C-style and functional casts.

### <a name="Res-casts-const"></a>ES.50: Don't cast away `const`

##### Reason

It makes a lie out of `const`.

##### Note

Usually the reason to "cast away `const`" is to allow the updating of some transient information of an otherwise immutable object.
Examples are caching, memoization, and precomputation.
Such examples are often handled as well or better using `mutable` or an indirection than with a `const_cast`.

##### Example

Consider keeping previously computed results around for a costly operation:

    int compute(int x); // compute a value for x; assume this to be costly

    class Cache {   // some type implementing a cache for an int->int operation
    public:
        pair<bool, int> find(int x) const;   // is there a value for x?
        void set(int x, int v);             // make y the value for x
        // ...
    private:
        // ...
    };

    class X {
    public:
        int get_val(int x)
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache.set(x, val); // insert value for x
            return val;
        }
        // ...
    private:
        Cache cache;
    };

Here, `get_val()` is logically constant, so we would like to make it a `const` member.
To do this we still need to mutate `cache`, so people sometimes resort to a `const_cast`:

    class X {   // Suspicious solution based on casting
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            const_cast<Cache&>(cache).set(x, val);   // ugly
            return val;
        }
        // ...
    private:
        Cache cache;
    };

Fortunately, there is a better solution:
State that `cache` is mutable even for a `const` object:

    class X {   // better solution
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache.set(x, val);
            return val;
        }
        // ...
    private:
        mutable Cache cache;
    };

An alternative solution would to store a pointer to the `cache`:

    class X {   // OK, but slightly messier solution
    public:
        int get_val(int x) const
        {
            auto p = cache->find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache->set(x, val);
            return val;
        }
        // ...
    private:
        unique_ptr<Cache> cache;
    };

That solution is the most flexible, but requires explicit construction and destruction of `*cache`
(most likely in the constructor and destructor of `X`).

In any variant, we must guard against data races on the `cache` in multithreaded code, possibly using a `std::mutex`.

##### Enforcement

Flag `const_cast`s. See also [Type.3: Don't use `const_cast` to cast away `const` (i.e., at all)](#Pro-type-constcast) for the related Profile.

### <a name="Res-range-checking"></a>ES.55: Avoid the need for range checking

##### Reason

Constructs that cannot overflow do not overflow (and usually run faster):

##### Example

    for (auto& x : v)      // print all elements of v
        cout << x << '\n';

    auto p = find(v, x);   // find x in v

##### Enforcement

Look for explicit range checks and heuristically suggest alternatives.

### <a name="Res-move"></a>ES.56: Write `std::move()` only when you need to explicitly move an object to another scope

##### Reason

We move, rather than copy, to avoid duplication and for improved performance.

A move typically leaves behind an empty object ([C.64](#Rc-move-semantic)), which can be surprising or even dangerous, so we try to avoid moving from lvalues (they might be accessed later).

##### Notes

Moving is done implicitly when the source is an rvalue (e.g., value in a `return` treatment or a function result), so don't pointlessly complicate code in those cases by writing `move` explicitly. Instead, write short functions that return values, and both the function's return and the caller's accepting of the return will be optimized naturally.

In general, following the guidelines in this document (including not making variables' scopes needlessly large, writing short functions that return values, returning local variables) help eliminate most need for explicit `std::move`.

Explicit `move` is needed to explicitly move an object to another scope, notably to pass it to a "sink" function and in the implementations of the move operations themselves (move constructor, move assignment operator) and swap operations.

##### Example, bad

    void sink(X&& x);   // sink takes ownership of x

    void user()
    {
        X x;
        // error: cannot bind an lvalue to a rvalue reference
        sink(x);
        // OK: sink takes the contents of x, x must now be assumed to be empty
        sink(std::move(x));

        // ...

        // probably a mistake
        use(x);
    }

Usually, a `std::move()` is used as an argument to a `&&` parameter.
And after you do that, assume the object has been moved from (see [C.64](#Rc-move-semantic)) and don't read its state again until you first set it to a new value.

    void f() {
        string s1 = "supercalifragilisticexpialidocious";

        string s2 = s1;             // ok, takes a copy
        assert(s1 == "supercalifragilisticexpialidocious");  // ok

        // bad, if you want to keep using s1's value
        string s3 = move(s1);

        // bad, assert will likely fail, s1 likely changed
        assert(s1 == "supercalifragilisticexpialidocious");
    }

##### Example

    void sink(unique_ptr<widget> p);  // pass ownership of p to sink()

    void f() {
        auto w = make_unique<widget>();
        // ...
        sink(std::move(w));               // ok, give to sink()
        // ...
        sink(w);    // Error: unique_ptr is carefully designed so that you cannot copy it
    }

##### Notes

`std::move()` is a cast to `&&` in disguise; it doesn't itself move anything, but marks a named object as a candidate that can be moved from.
The language already knows the common cases where objects can be moved from, especially when returning values from functions, so don't complicate code with redundant `std::move()`'s.

Never write `std::move()` just because you've heard "it's more efficient."
In general, don't believe claims of "efficiency" without data (???).
In general, don't complicate your code without reason (??)

##### Example, bad

    vector<int> make_vector() {
        vector<int> result;
        // ... load result with data
        return std::move(result);       // bad; just write "return result;"
    }

Never write `return move(local_variable);`, because the language already knows the variable is a move candidate.
Writing `move` in this code won't help, and can actually be detrimental because on some compilers it interferes with RVO (the return value optimization) by creating an additional reference alias to the local variable.


##### Example, bad

    vector<int> v = std::move(make_vector());   // bad; the std::move is entirely redundant

Never write `move` on a returned value such as `x = move(f());` where `f` returns by value.
The language already knows that a returned value is a temporary object that can be moved from.

##### Example

    void mover(X&& x) {
        call_something(std::move(x));         // ok
        call_something(std::forward<X>(x));   // bad, don't std::forward an rvalue reference
        call_something(x);                    // suspicious, why not std::move?
    }

    template<class T>
    void forwarder(T&& t) {
        call_something(std::move(t));         // bad, don't std::move a forwarding reference
        call_something(std::forward<T>(t));   // ok
        call_something(t);                    // suspicious, why not std::forward?
    }

##### Enforcement

* Flag use of `std::move(x)` where `x` is an rvalue or the language will already treat it as an rvalue, including `return std::move(local_variable);` and `std::move(f())` on a function that returns by value.
* Flag functions taking an `S&&` parameter if there is no `const S&` overload to take care of lvalues.
* Flag a `std::move`s argument passed to a parameter, except when the parameter type is one of the following: an `X&&` rvalue reference; a `T&&` forwarding reference where `T` is a template parameter type; or by value and the type is move-only.
* Flag when `std::move` is applied to a forwarding reference (`T&&` where `T` is a template parameter type). Use `std::forward` instead.
* Flag when `std::move` is applied to other than an rvalue reference. (More general case of the previous rule to cover the non-forwarding cases.)
* Flag when `std::forward` is applied to an rvalue reference (`X&&` where `X` is a concrete type). Use `std::move` instead.
* Flag when `std::forward` is applied to other than a forwarding reference. (More general case of the previous rule to cover the non-moving cases.)
* Flag when an object is potentially moved from and the next operation is a `const` operation; there should first be an intervening non-`const` operation, ideally assignment, to first reset the object's value.

### <a name="Res-new"></a>ES.60: Avoid `new` and `delete` outside resource management functions

##### Reason

Direct resource management in application code is error-prone and tedious.

##### Note

also known as "No naked `new`!"

##### Example, bad

    void f(int n)
    {
        auto p = new X[n];   // n default constructed Xs
        // ...
        delete[] p;
    }

There can be code in the `...` part that causes the `delete` never to happen.

**See also**: [R: Resource management](#S-resource).

##### Enforcement

Flag naked `new`s and naked `delete`s.

### <a name="Res-del"></a>ES.61: Delete arrays using `delete[]` and non-arrays using `delete`

##### Reason

That's what the language requires and mistakes can lead to resource release errors and/or memory corruption.

##### Example, bad

    void f(int n)
    {
        auto p = new X[n];   // n default constructed Xs
        // ...
        delete p;   // error: just delete the object p, rather than delete the array p[]
    }

##### Note

This example not only violates the [no naked `new` rule](#Res-new) as in the previous example, it has many more problems.

##### Enforcement

* if the `new` and the `delete` is in the same scope, mistakes can be flagged.
* if the `new` and the `delete` are in a constructor/destructor pair, mistakes can be flagged.

### <a name="Res-arr2"></a>ES.62: Don't compare pointers into different arrays

##### Reason

The result of doing so is undefined.

##### Example, bad

    void f(int n)
    {
        int a1[7];
        int a2[9];
        if (&a1[5] < &a2[7]) {}       // bad: undefined
        if (0 < &a1[5] - &a2[7]) {}   // bad: undefined
    }

##### Note

This example has many more problems.

##### Enforcement

???

### <a name="Res-slice"></a>ES.63: Don't slice

##### Reason

Slicing -- that is, copying only part of an object using assignment or initialization -- most often leads to errors because
the object was meant to be considered as a whole.
In the rare cases where the slicing was deliberate the code can be surprising.

##### Example

    class Shape { /* ... */ };
    class Circle : public Shape { /* ... */ Point c; int r; };

    Circle c {{0, 0}, 42};
    Shape s {c};    // copy Shape part of Circle

The result will be meaningless because the center and radius will not be copied from `c` into `s`.
The first defense against this is to [define the base class `Shape` not to allow this](#Rc-copy-virtual).

##### Alternative

If you mean to slice, define an explicit operation to do so.
This saves readers from confusion.
For example:

    class Smiley : public Circle {
        public:
        Circle copy_circle();
        // ...
    };

    Smiley sm { /* ... */ };
    Circle c1 {sm};  // ideally prevented by the definition of Circle
    Circle c2 {sm.copy_circle()};

##### Enforcement

Warn against slicing.

## <a name="SS-numbers"></a>Arithmetic

### <a name="Res-mix"></a>ES.100: Don't mix signed and unsigned arithmetic

##### Reason

Avoid wrong results.

##### Example

    int x = -3;
    unsigned int y = 7;

    cout << x - y << '\n';  // unsigned result, possibly 4294967286
    cout << x + y << '\n';  // unsigned result: 4
    cout << x * y << '\n';  // unsigned result, possibly 4294967275

It is harder to spot the problem in more realistic examples.

##### Note

Unfortunately, C++ uses signed integers for array subscripts and the standard library uses unsigned integers for container subscripts.
This precludes consistency.

##### Enforcement

Compilers already know and sometimes warn.

### <a name="Res-unsigned"></a>ES.101: Use unsigned types for bit manipulation

##### Reason

Unsigned types support bit manipulation without surprises from sign bits.

##### Example

    unsigned char x = 0b1010'1010;
    unsigned char y = ~x;   // y == 0b0101'0101;

##### Note

Unsigned types can also be useful for modulo arithmetic.
However, if you want modulo arithmetic add
comments as necessary noting the reliance on wraparound behavior, as such code
can be surprising for many programmers.

##### Enforcement

* Just about impossible in general because of the use of unsigned subscripts in the standard library
* ???

### <a name="Res-signed"></a>ES.102: Use signed types for arithmetic

##### Reason

Because most arithmetic is assumed to be signed;
`x-y` yields a negative number when `y>x` except in the rare cases where you really want modulo arithmetic.

##### Example

Unsigned arithmetic can yield surprising results if you are not expecting it.
This is even more true for mixed signed and unsigned arithmetic.

    template<typename T, typename T2>
    T subtract(T x, T2 y)
    {
        return x-y;
    }

    void test()
    {
        int s = 5;
        unsigned int us = 5;
        cout << subtract(s, 7) << '\n';     // -2
        cout << subtract(us, 7u) << '\n';   // 4294967294
        cout << subtract(s, 7u) << '\n';    // -2
        cout << subtract(us, 7) << '\n';    // 4294967294
        cout << subtract(s, us+2) << '\n';  // -2
        cout << subtract(us, s+2) << '\n';  // 4294967294
    }

Here we have been very explicit about what's happening,
but if you had seen `us-(s+2)` or `s+=2; ... us-s`, would you reliably have suspected that the result would print as `4294967294`?

##### Exception

Use unsigned types if you really want modulo arithmetic - add
comments as necessary noting the reliance on overflow behavior, as such code
is going to be surprising for many programmers.

##### Example

The standard library uses unsigned types for subscripts.
The build-in array uses signed types for subscripts.
This makes surprises (and bugs) inevitable.

    int a[10];
    for (int i=0; i < 10; ++i) a[i]=i;
    vector<int> v(10);
    // compares signed to unsigned; some compilers warn
    for (int i=0; v.size() < 10; ++i) v[i]=i;

    int a2[-2];         // error: negative size

    // OK, but the number of ints (4294967294) is so large that we should get an exception
    vector<int> v2(-2);

##### Enforcement

* Flag mixed signed and unsigned arithmetic
* Flag results of unsigned arithmetic assigned to or printed as signed.
* Flag unsigned literals (e.g. `-2`) used as container subscripts.

### <a name="Res-overflow"></a>ES.103: Don't overflow

##### Reason

Overflow usually makes your numeric algorithm meaningless.
Incrementing a value beyond a maximum value can lead to memory corruption and undefined behavior.

##### Example, bad

    int a[10];
    a[10] = 7;   // bad

    int n = 0;
    while (n++ < 10)
        a[n - 1] = 9; // bad (twice)

##### Example, bad

    int n = numeric_limits<int>::max();
    int m = n + 1;   // bad

##### Example, bad

    int area(int h, int w) { return h * w; }

    auto a = area(10'000'000, 100'000'000);   // bad

##### Exception

Use unsigned types if you really want modulo arithmetic.

**Alternative**: For critical applications that can afford some overhead, use a range-checked integer and/or floating-point type.

##### Enforcement

???

### <a name="Res-underflow"></a>ES.104: Don't underflow

##### Reason

Decrementing a value beyond a minimum value can lead to memory corruption and undefined behavior.

##### Example, bad

    int a[10];
    a[-2] = 7;   // bad

    int n = 101;
    while (n--)
        a[n - 1] = 9;   // bad (twice)

##### Exception

Use unsigned types if you really want modulo arithmetic.

##### Enforcement

???

### <a name="Res-zero"></a>ES.105: Don't divide by zero

##### Reason

The result is undefined and probably a crash.

##### Note

This also applies to `%`.

##### Example; bad

    double divide(int a, int b) {
        // BAD, should be checked (e.g., in a precondition)
        return a / b;
    }

##### Example; good

    double divide(int a, int b) {
        // good, address via precondition (and replace with contracts once C++ gets them)
        Expects(b != 0);
        return a / b;
    }

    double divide(int a, int b) {
        // good, address via check
        return b ? a / b : quiet_NaN<double>();
    }

**Alternative**: For critical applications that can afford some overhead, use a range-checked integer and/or floating-point type.

##### Enforcement

* Flag division by an integral value that could be zero

