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

  


