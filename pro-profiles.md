# <span id="S-profile"></span>Pro: Profiles

A "profile" is a set of deterministic and portably enforceable subset rules (i.e., restrictions) that are designed to achieve a specific guarantee. "Deterministic" means they require only local analysis and could be implemented in a compiler (though they don't need to be). "Portably enforceable" means they are like language rules, so programmers can count on enforcement tools giving the same answer for the same code.

Code written to be warning-free using such a language profile is considered to conform to the profile. Conforming code is considered to be safe by construction with regard to the safety properties targeted by that profile. Conforming code will not be the root cause of errors for that property, although such errors may be introduced into a program by other code, libraries or the external environment. A profile may also introduce additional library types to ease conformance and encourage correct code.

Profiles summary:

* [Pro.type: Type safety](#SS-type)
* [Pro.bounds: Bounds safety](#SS-bounds)
* [Pro.lifetime: Lifetime safety](#SS-lifetime)

In the future, we expect to define many more profiles and add more checks to existing profiles.
Candidates include:

* narrowing arithmetic promotions/conversions (likely part of a separate safe-arithmetic profile)
* arithmetic cast from negative floating point to unsigned integral type (ditto)
* selected undefined behavior: ??? start with Gaby's UB list
* selected unspecified behavior: ??? a portability concern?
* `const` violations

To suppress enforcement of a profile check, place a `suppress` annotation on a language contract. For example:

    [[suppress(bounds)]] char* raw_find(char* p, int n, char x)    // find x in p[0]..p[n-1]
    {
        // ...
    }

Now `raw_find()` can scramble memory to its heart's content.
Obviously, suppression should be very rare.

## <span id="SS-type"></span>Pro.safety: Type safety profile

This profile makes it easier to construct code that uses types correctly and avoids inadvertent type punning.
It does so by focusing on removing the primary sources of type violations, including unsafe uses of casts and unions.

For the purposes of this section,
type-safety is defined to be the property that a variable is not used in a way that doesn't obey the rules for the type of its definition.
Memory accessed as a type `T` should not be valid memory that actually contains an object of an unrelated type `U`.
Note that the safety is intended to be complete when combined also with [Bounds safety](#SS-bounds) and [Lifetime safety](#SS-lifetime).

An implementation of this profile shall recognize the following patterns in source code as non-conforming and issue a diagnostic.

Type safety profile summary:

* [Type.1: Don't use `reinterpret_cast`](#Pro-type-reinterpretcast)
* [Type.2: Don't use `static_cast` downcasts. Use `dynamic_cast` instead](#Pro-type-downcast)
* [Type.3: Don't use `const_cast` to cast away `const` (i.e., at all)](#Pro-type-constcast)
* [Type.4: Don't use C-style `(T)expression` casts that would perform a `static_cast` downcast, `const_cast`, or `reinterpret_cast`](#Pro-type-cstylecast)
* [Type.4.1: Don't use `T(expression)` for casting](#Pro-fct-style-cast)
* [Type.5: Don't use a variable before it has been initialized](#Pro-type-init)
* [Type.6: Always initialize a member variable](#Pro-type-memberinit)
* [Type.7: Avoid accessing members of raw unions. Prefer `variant` instead](#Pro-fct-style-cast)
* [Type.8: Avoid reading from varargs or passing vararg arguments. Prefer variadic template parameters instead](#Pro-type-varargs)

### <span id="Pro-type-reinterpretcast"></span>Type.1: Don't use `reinterpret_cast`.

##### Reason

Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.

##### Example, bad

    std::string s = "hello world";
    double* p = reinterpret_cast<double*>(&s); // BAD

##### Enforcement

Issue a diagnostic for any use of `reinterpret_cast`. To fix: Consider using a `variant` instead.

### <span id="Pro-type-downcast"></span>Type.2: Don't use `static_cast` downcasts. Use `dynamic_cast` instead.

##### Reason

Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.

##### Example, bad

    class Base { public: virtual ~Base() = 0; };

    class Derived1 : public Base { };

    class Derived2 : public Base {
        std::string s;
    public:
        std::string get_s() { return s; }
    };

    Derived1 d1;
    Base* p1 = &d1; // ok, implicit conversion to pointer to Base is fine

    // BAD, tries to treat d1 as a Derived2, which it is not
    Derived2* p2 = static_cast<Derived2*>(p1);
    // tries to access d1's nonexistent string member, instead sees arbitrary bytes near d1
    cout << p2->get_s();

##### Example, bad

    struct Foo { int a, b; };
    struct Foobar : Foo { int bar; };

    void use(int i, Foo& x)
    {
        if (0 < i) {
            Foobar& x1 = dynamic_cast<Foobar&>(x);  // error: Foo is not polymorphic
            Foobar& x2 = static_cast<Foobar&>(x);   // bad
            // ...
        }
        // ...
    }

    // ...

    use(99, *new Foo{1, 2});  // not a Foobar

If a class hierarchy isn't polymorphic, avoid casting.
It is entirely unsafe.
Look for a better design.
See also [C.146](#Rh-dynamic_cast).

##### Enforcement

Issue a diagnostic for any use of `static_cast` to downcast, meaning to cast from a pointer or reference to `X` to a pointer or reference to a type that is not `X` or an accessible base of `X`. To fix: If this is a downcast or cross-cast then use a `dynamic_cast` instead, otherwise consider using a `variant` instead.

### <span id="Pro-type-constcast"></span>Type.3: Don't use `const_cast` to cast away `const` (i.e., at all).

##### Reason

Casting away `const` is a lie. If the variable is actually declared `const`, it's a lie punishable by undefined behavior.

##### Example, bad

    void f(const int& i)
    {
        const_cast<int&>(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // silent side effect
    f(j); // undefined behavior

##### Example

Sometimes you may be tempted to resort to `const_cast` to avoid code duplication, such as when two accessor functions that differ only in `const`-ness have similar implementations. For example:

    class Bar;

    class Foo {
    public:
        // BAD, duplicates logic
        Bar& get_bar() {
            /* complex logic around getting a non-const reference to my_bar */
        }

        const Bar& get_bar() const {
            /* same complex logic around getting a const reference to my_bar */
        }
    private:
        Bar my_bar;
    };

Instead, prefer to share implementations. Normally, you can just have the non-`const` function call the `const` function. However, when there is complex logic this can lead to the following pattern that still resorts to a `const_cast`:

    class Foo {
    public:
        // not great, non-const calls const version but resorts to const_cast
        Bar& get_bar() {
            return const_cast<Bar&>(static_cast<const Foo&>(*this).get_bar());
        }
        const Bar& get_bar() const {
            /* the complex logic around getting a const reference to my_bar */
        }
    private:
        Bar my_bar;
    };

Although this pattern is safe when applied correctly, because the caller must have had a non-`const` object to begin with, it's not ideal because the safety is hard to enforce automatically as a checker rule.

Instead, prefer to put the common code in a common helper function -- and make it a template so that it deduces `const`. This doesn't use any `const_cast` at all:

    class Foo {
    public:                         // good
              Bar& get_bar()       { return get_bar_impl(*this); }
        const Bar& get_bar() const { return get_bar_impl(*this); }
    private:
        Bar my_bar;

        template<class T>           // good, deduces whether T is const or non-const
        static auto get_bar_impl(T& t) -> decltype(t.get_bar())
            { /* the complex logic around getting a possibly-const reference to my_bar */ }
    };

##### Exception

You may need to cast away `const` when calling `const`-incorrect functions. Prefer to wrap such functions in inline `const`-correct wrappers to encapsulate the cast in one place.

##### See also: 

[ES.50, Don't cast away `const`](#Res-casts-const) for more discussion.

##### Enforcement

Issue a diagnostic for any use of `const_cast`. To fix: Either don't use the variable in a non-`const` way, or don't make it `const`.

### <span id="Pro-type-cstylecast"></span>Type.4: Don't use C-style `(T)expression` casts that would perform a `static_cast` downcast, `const_cast`, or `reinterpret_cast`.

##### Reason

Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.
Note that a C-style `(T)expression` cast means to perform the first of the following that is possible: a `const_cast`, a `static_cast`, a `static_cast` followed by a `const_cast`, a `reinterpret_cast`, or a `reinterpret_cast` followed by a `const_cast`. This rule bans `(T)expression` only when used to perform an unsafe cast.

##### Example, bad

    std::string s = "hello world";
    double* p0 = (double*)(&s); // BAD

    class Base { public: virtual ~Base() = 0; };

    class Derived1 : public Base { };

    class Derived2 : public Base {
        std::string s;
    public:
        std::string get_s() { return s; }
    };

    Derived1 d1;
    Base* p1 = &d1; // ok, implicit conversion to pointer to Base is fine

    // BAD, tries to treat d1 as a Derived2, which it is not
    Derived2* p2 = (Derived2*)(p1);
    // tries to access d1's nonexistent string member, instead sees arbitrary bytes near d1
    cout << p2->get_s();

    void f(const int& i) {
        (int&)(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // silent side effect
    f(j); // undefined behavior

##### Enforcement

Issue a diagnostic for any use of a C-style `(T)expression` cast that would invoke a `static_cast` downcast, `const_cast`, or `reinterpret_cast`. To fix: Use a `dynamic_cast`, `const`-correct declaration, or `variant`, respectively.

### <span id="Pro-fct-style-cast"></span>Type.4.1: Don't use `T(expression)` for casting.

##### Reason

If `e` is of a built-in type, `T(e)` is equivalent to the error-prone `(T)e`.

##### Example, bad

    int* p = f(x);
    auto i = int(p);    // Potential damaging cast; don't or use `reinterpret_cast`

    short s = short(i); // potentially narrowing; don't or use `narrow` or `narrow_cast`

##### Note

The {}-syntax makes the desire for construction explicit and doesn't allow narrowing

    f(Foo{bar});

##### Enforcement

Flag `T(e)` if used for `e` of a built-in type.

### <span id="Pro-type-init"></span>Type.5: Don't use a variable before it has been initialized.

[ES.20: Always initialize an object](#Res-always) is required.

### <span id="Pro-type-memberinit"></span>Type.6: Always initialize a member variable.

##### Reason

Before a variable has been initialized, it does not contain a deterministic valid value of its type. It could contain any arbitrary bit pattern, which could be different on each call.

##### Example

    struct X { int i; };

    X x;
    use(x); // BAD, x has not been initialized

    X x2{}; // GOOD
    use(x2);

##### Enforcement

* Issue a diagnostic for any constructor of a non-trivially-constructible type that does not initialize all member variables. To fix: Write a data member initializer, or mention it in the member initializer list.
* Issue a diagnostic when constructing an object of a trivially constructible type without `()` or `{}` to initialize its members. To fix: Add `()` or `{}`.

### <span id="Pro-type-unions"></span>Type.7: Avoid accessing members of raw unions. Prefer `variant` instead.

##### Reason

Reading from a union member assumes that member was the last one written, and writing to a union member assumes another member with a nontrivial destructor had its destructor called. This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

##### Example

    union U { int i; double d; };

    U u;
    u.i = 42;
    use(u.d); // BAD, undefined

    variant<int, double> u;
    u = 42; // u now contains int
    use(u.get<int>()); // ok
    use(u.get<double>()); // throws ??? update this when standardization finalizes the variant design

Note that just copying a union is not type-unsafe, so safe code can pass a union from one piece of unsafe code to another.

##### Enforcement

* Issue a diagnostic for accessing a member of a union. To fix: Use a `variant` instead.

### <span id="Pro-type-varargs"></span>Type.8: Avoid reading from varargs or passing vararg arguments. Prefer variadic template parameters instead.

##### Reason

Reading from a vararg assumes that the correct type was actually passed. Passing to varargs assumes the correct type will be read. This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

##### Example

    int sum(...) {
        // ...
        while (/*...*/)
            result += va_arg(list, int); // BAD, assumes it will be passed ints
        // ...
    }

    sum(3, 2); // ok
    sum(3.14159, 2.71828); // BAD, undefined

    template<class ...Args>
    auto sum(Args... args) { // GOOD, and much more flexible
        return (... + args); // note: C++17 "fold expression"
    }

    sum(3, 2); // ok: 5
    sum(3.14159, 2.71828); // ok: ~5.85987

Note: Declaring a `...` parameter is sometimes useful for techniques that don't involve actual argument passing, notably to declare "take-anything" functions so as to disable "everything else" in an overload set or express a catchall case in a template metaprogram.

##### Enforcement

* Issue a diagnostic for using `va_list`, `va_start`, or `va_arg`. To fix: Use a variadic template parameter list instead.
* Issue a diagnostic for passing an argument to a vararg parameter of a function that does not offer an overload for a more specific type in the position of the vararg. To fix: Use a different function, or `[[suppress(types)]]`.

## <span id="SS-bounds"></span>Pro.bounds: Bounds safety profile

This profile makes it easier to construct code that operates within the bounds of allocated blocks of memory. It does so by focusing on removing the primary sources of bounds violations: pointer arithmetic and array indexing. One of the core features of this profile is to restrict pointers to only refer to single objects, not arrays.

For the purposes of this document, bounds-safety is defined to be the property that a program does not use a variable to access memory outside of the range that was allocated and assigned to that variable. (Note that the safety is intended to be complete when combined also with [Type safety](#SS-type) and [Lifetime safety](#SS-lifetime), which cover other unsafe operations that allow bounds violations, such as type-unsafe casts that 'widen' pointers.)

The following are under consideration but not yet in the rules below, and may be better in other profiles:

* ???

An implementation of this profile shall recognize the following patterns in source code as non-conforming and issue a diagnostic.

Bounds safety profile summary:

* [Bounds.1: Don't use pointer arithmetic. Use `span` instead](#Pro-bounds-arithmetic)
* [Bounds.2: Only index into arrays using constant expressions](#Pro-bounds-arrayindex)
* [Bounds.3: No array-to-pointer decay](#Pro-bounds-decay)
* [Bounds.4: Don't use standard library functions and types that are not bounds-checked](#Pro-bounds-stdlib)


### <span id="Pro-bounds-arithmetic"></span>Bounds.1: Don't use pointer arithmetic. Use `span` instead.

##### Reason

Pointers should only refer to single objects, and pointer arithmetic is fragile and easy to get wrong. `span` is a bounds-checked, safe type for accessing arrays of data.

##### Example, bad

    void f(int* p, int count)
    {
        if (count < 2) return;

        int* q = p + 1; // BAD

        ptrdiff_t d;
        int n;
        d = (p - &n); // OK
        d = (q - p); // OK

        int n = *p++; // BAD

        if (count < 6) return;

        p[4] = 1; // BAD

        p[count - 1] = 2; // BAD

        use(&p[0], 3); // BAD
    }

##### Example, good

    void f(span<int> a) // BETTER: use span in the function declaration
    {
        if (a.length() < 2) return;

        int n = a[0]; // OK

        span<int> q = a.subspan(1); // OK

        if (a.length() < 6) return;

        a[4] = 1; // OK

        a[count - 1] = 2; // OK

        use(a.data(), 3); // OK
    }

##### Enforcement

Issue a diagnostic for any arithmetic operation on an expression of pointer type that results in a value of pointer type.

### <span id="Pro-bounds-arrayindex"></span>Bounds.2: Only index into arrays using constant expressions.

##### Reason

Dynamic accesses into arrays are difficult for both tools and humans to validate as safe. `span` is a bounds-checked, safe type for accessing arrays of data. `at()` is another alternative that ensures single accesses are bounds-checked. If iterators are needed to access an array, use the iterators from a `span` constructed over the array.

##### Example, bad

    void f(array<int, 10> a, int pos)
    {
        a[pos / 2] = 1; // BAD
        a[pos - 1] = 2; // BAD
        a[-1] = 3;    // BAD -- no replacement, just don't do this
        a[10] = 4;    // BAD -- no replacement, just don't do this
    }

##### Example, good

    // ALTERNATIVE A: Use a span

    // A1: Change parameter type to use span
    void f1(span<int, 10> a, int pos)
    {
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }

    // A2: Add local span and use that
    void f2(array<int, 10> arr, int pos)
    {
        span<int> a = {arr, pos}
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }

    // ALTERNATIVE B: Use at() for access
    void f3(array<int, 10> a, int pos)
    {
        at(a, pos / 2) = 1; // OK
        at(a, pos - 1) = 2; // OK
    }

##### Example, bad

    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            arr[i] = i; // BAD, cannot use non-constant indexer
    }

##### Example, good

    // ALTERNATIVE A: Use a span
    void f1()
    {
        int arr[COUNT];
        span<int> av = arr;
        for (int i = 0; i < COUNT; ++i)
            av[i] = i;
    }
    
    // ALTERNATIVE Aa: Use a span and range-for
    void f1a()
    {
         int arr[COUNT];
         span<int, COUNT> av = arr;
         int i = 0;
         for (auto& e : av)
             e = i++;
    }

    // ALTERNATIVE B: Use at() for access
    void f2()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            at(arr, i) = i;
    }

##### Enforcement

Issue a diagnostic for any indexing expression on an expression or variable of array type (either static array or `std::array`) where the indexer is not a compile-time constant expression.

Issue a diagnostic for any indexing expression on an expression or variable of array type (either static array or `std::array`) where the indexer is not a value between `0` or and the upper bound of the array.

**Rewrite support**: Tooling can offer rewrites of array accesses that involve dynamic index expressions to use `at()` instead:

    static int a[10];

    void f(int i, int j)
    {
        a[i + j] = 12;      // BAD, could be rewritten as ...
        at(a, i + j) = 12;  // OK -- bounds-checked
    }

### <span id="Pro-bounds-decay"></span>Bounds.3: No array-to-pointer decay.

##### Reason

Pointers should not be used as arrays. `span` is a bounds-checked, safe alternative to using pointers to access arrays.

##### Example, bad

    void g(int* p, size_t length);

    void f()
    {
        int a[5];
        g(a, 5);        // BAD
        g(&a[0], 1);    // OK
    }

##### Example, good

    void g(int* p, size_t length);
    void g1(span<int> av); // BETTER: get g() changed.

    void f()
    {
        int a[5];
        span<int> av = a;

        g(av.data(), av.length());   // OK, if you have no choice
        g1(a);                       // OK -- no decay here, instead use implicit span ctor
    }

##### Enforcement

Issue a diagnostic for any expression that would rely on implicit conversion of an array type to a pointer type.

### <span id="Pro-bounds-stdlib"></span>Bounds.4: Don't use standard library functions and types that are not bounds-checked.

##### Reason

These functions all have bounds-safe overloads that take `span`. Standard types such as `vector` can be modified to perform bounds-checks under the bounds profile (in a compatible way, such as by adding contracts), or used with `at()`.

##### Example, bad

    void f()
    {
        array<int, 10> a, b;
        memset(a.data(), 0, 10);         // BAD, and contains a length error (length = 10 * sizeof(int))
        memcmp(a.data(), b.data(), 10);  // BAD, and contains a length error (length = 10 * sizeof(int))
    }

Also, `std::array<>::fill()` or `std::fill()` or even an empty initializer are better candidate than `memset()`.

##### Example, good

    void f()
    {
        array<int, 10> a, b, c{};       // c is initialized to zero
        a.fill(0);
        fill(b.begin(), b.end(), 0);    // std::fill()
        fill(b, 0);                     // std::fill() + Ranges TS

        if ( a == b ) {
          // ...
        }
    }

##### Example

If code is using an unmodified standard library, then there are still workarounds that enable use of `std::array` and `std::vector` in a bounds-safe manner. Code can call the `.at()` member function on each class, which will result in an `std::out_of_range` exception being thrown. Alternatively, code can call the `at()` free function, which will result in fail-fast (or a customized action) on a bounds violation.

    void f(std::vector<int>& v, std::array<int, 12> a, int i)
    {
        v[0] = a[0];        // BAD
        v.at(0) = a[0];     // OK (alternative 1)
        at(v, 0) = a[0];    // OK (alternative 2)

        v.at(0) = a[i];     // BAD
        v.at(0) = a.at(i);  // OK (alternative 1)
        v.at(0) = at(a, i); // OK (alternative 2)
    }

##### Enforcement

* Issue a diagnostic for any call to a standard library function that is not bounds-checked. ??? insert link to a list of banned functions

**TODO Notes**:

* Impact on the standard library will require close coordination with WG21, if only to ensure compatibility even if never standardized.
* We are considering specifying bounds-safe overloads for stdlib (especially C stdlib) functions like `memcmp` and shipping them in the GSL.
* For existing stdlib functions and types like `vector` that are not fully bounds-checked, the goal is for these features to be bounds-checked when called from code with the bounds profile on, and unchecked when called from legacy code, possibly using contracts (concurrently being proposed by several WG21 members).

## <span id="SS-lifetime"></span>Pro.lifetime: Lifetime safety profile

See /docs folder for the initial design. The formal rules are in progress (as of March 2017).
 
