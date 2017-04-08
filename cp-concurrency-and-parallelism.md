# <a name="S-concurrency"></a>CP: Concurrency and Parallelism

We often want our computers to do many tasks at the same time (or at least make them appear to do them at the same time).
The reasons for doing so varies (e.g., wanting to wait for many events using only a single processor, processing many data streams simultaneously, or utilizing many hardware facilities)
and so does the basic facilities for expressing concurrency and parallelism.
Here, we articulate a few general principles and rules for using the ISO standard C++ facilities for expressing basic concurrency and parallelism.

The core machine support for concurrent and parallel programming is the thread.
Threads allow you to run multiple instances of your program independently, while sharing
the same memory. Concurrent programming is tricky for many reasons, most
importantly that it is undefined behavior to read data in one thread after it
was written by another thread, if there is no proper synchronization between
those threads. Making existing single-threaded code execute concurrently can be
as trivial as adding `std::async` or `std::thread` strategically, or it can
necessitate a full rewrite, depending on whether the original code was written
in a thread-friendly way.

The concurrency/parallelism rules in this document are designed with three goals
in mind:

* To help you write code that is amenable to being used in a threaded
  environment
* To show clean, safe ways to use the threading primitives offered by the
  standard library
* To offer guidance on what to do when concurrency and parallelism aren't giving
  you the performance gains you need

It is also important to note that concurrency in C++ is an unfinished
story. C++11 introduced many core concurrency primitives, C++14 improved on
them, and it seems that there is much interest in making the writing of
concurrent programs in C++ even easier. We expect some of the library-related
guidance here to change significantly over time.

This section needs a lot of work (obviously).
Please note that we start with rules for relative non-experts.
Real experts must wait a bit;
contributions are welcome,
but please think about the majority of programmers who are struggling to get their concurrent programs correct and performant.

Concurrency and parallelism rule summary:

* [CP.1: Assume that your code will run as part of a multi-threaded program](#Rconc-multi)
* [CP.2: Avoid data races](#Rconc-races)
* [CP.3: Minimize explicit sharing of writable data](#Rconc-data)
* [CP.4: Think in terms of tasks, rather than threads](#Rconc-task)
* [CP.8: Don't try to use `volatile` for synchronization](#Rconc-volatile)
* [CP.9: Whenever feasible use tools to validate your concurrent code](#Rconc-tools)

See also:

* [CP.con: Concurrency](#SScp-con)
* [CP.par: Parallelism](#SScp-par)
* [CP.mess: Message passing](#SScp-mess)
* [CP.vec: Vectorization](#SScp-vec)
* [CP.free: Lock-free programming](#SScp-free)
* [CP.etc: Etc. concurrency rules](#SScp-etc)

### <a name="Rconc-multi"></a>CP.1: Assume that your code will run as part of a multi-threaded program

##### Reason

It is hard to be certain that concurrency isn't used now or will be sometime in the future.
Code gets re-used.
Libraries using threads may be used from some other part of the program.
Note that this applies most urgently to library code and least urgently to stand-alone applications.
However, thanks to the magic of cut-and-paste, code fragments can turn up in unexpected places.

##### Example

    double cached_computation(double x)
    {
        static double cached_x = 0.0;
        static double cached_result = COMPUTATION_OF_ZERO;
        double result;

        if (cached_x == x)
            return cached_result;
        result = computation(x);
        cached_x = x;
        cached_result = result;
        return result;
    }

Although `cached_computation` works perfectly in a single-threaded environment, in a multi-threaded environment the two `static` variables result in data races and thus undefined behavior.
>>>>>>> a47e5a627f7b2ef31767de0251fd8958c85a16ee

There are several ways that this example could be made safe for a multi-threaded environment:

* Delegate concurrency concerns upwards to the caller.
<<<<<<< HEAD
* Mark the
  `static`
  variables as
  `thread_local`
  \(which might make caching less effective\).
* Implement concurrency control, for example, protecting the two
  `static`
  variables with a
  `static`
  lock \(which might reduce performance\).
=======
* Mark the `static` variables as `thread_local` (which might make caching less effective).
* Implement concurrency control, for example, protecting the two `static` variables with a `static` lock (which might reduce performance).
>>>>>>> a47e5a627f7b2ef31767de0251fd8958c85a16ee
* Have the caller provide the memory to be used for the cache, thereby delegating both memory allocation and concurrency concerns upwards to the caller.
* Refuse to build and/or run in a multi-threaded environment.
* Provide two implementations, one which is used in single-threaded environments and another which is used in multi-threaded environments.

<<<<<<< HEAD
##### Exception {#exception-43}

Code that is never run in a multi-threaded environment.

Be careful: there are many examples where code that was “known” to never run in a multi-threaded program was run as part of a multi-threaded program. Often years later. Typically, such programs lead to a painful effort to remove data races. Therefore, code that is never intended to run in a multi-threaded environment should be clearly labeled as such and ideally come with compile or run-time enforcement mechanisms to catch those usage bugs early.

### CP.2: Avoid data races {#cp2-avoid-data-races}

##### Reason {#reason-260}

Unless you do, nothing is guaranteed to work and subtle errors will persist.

##### Note {#note-209}

In a nutshell, if two threads can access the same object concurrently \(without synchronization\), and at least one is a writer \(performing a non-`const`operation\), you have a data race. For further information of how to use synchronization well to eliminate data races, please consult a good book about concurrency.

##### Example, bad {#example-bad-100}

There are many examples of data races that exist, some of which are running in production software at this very moment. One very simple example:

```
int get_id() {
  static int id = 1;
  return id++;
}

```

The increment here is an example of a data race. This can go wrong in many ways, including:

* Thread A loads the value of
  `id`
  , the OS context switches A out for some period, during which other threads create hundreds of IDs. Thread A is then allowed to run again, and
  `id`
  is written back to that location as A’s read of
  `id`
  plus one.
* Thread A and B load
  `id`
  and increment it simultaneously. They both get the same ID.

Local static variables are a common source of data races.

##### Example, bad: {#example-bad-101}

```
void f(fstream
&
  fs, regex pat)
{
    array
<
double, max
>
 buf;
    int sz = read_vec(fs, buf, max);            // read from fs into buf
    gsl::span
<
double
>
 s {buf};
    // ...
    auto h1 = async([
&
]{ sort(par, s); });     // spawn a task to sort
    // ...
    auto h2 = async([
&
]{ return find_all(buf, sz, pat); });   // span a task to find matches
    // ...
}

```

Here, we have a \(nasty\) data race on the elements of`buf`\(`sort`will both read and write\). All data races are nasty. Here, we managed to get a data race on data on the stack. Not all data races are as easy to spot as this one.

##### Example, bad: {#example-bad-102}

```
// code not controlled by a lock

unsigned val;

if (val 
<
 5) {
    // ... other thread can change val here ...
    switch (val) {
    case 0: // ...
    case 1: // ...
    case 2: // ...
    case 3: // ...
    case 4: // ...
    }
}

```

Now, a compiler that does not know that`val`can change will most likely implement that`switch`using a jump table with five entries. Then, a`val`outside the`[0..4]`range will cause a jump to an address that could be anywhere in the program, and execution would proceed there. Really, “all bets are off” if you get a data race. Actually, it can be worse still: by looking at the generated code you may be able to determine where the stray jump will go for a given value; this can be a security risk.

##### Enforcement {#enforcement-240}

Some is possible, do at least something. There are commercial and open-source tools that try to address this problem, but be aware that solutions have costs and blind spots. Static tools often have many false positives and run-time tools often have a significant cost. We hope for better tools. Using multiple tools can catch more problems than a single one.
=======
##### Exception

Code that is never run in a multi-threaded environment.

Be careful: there are many examples where code that was "known" to never run in a multi-threaded program
was run as part of a multi-threaded program. Often years later.
Typically, such programs lead to a painful effort to remove data races.
Therefore, code that is never intended to run in a multi-threaded environment should be clearly labeled as such and ideally come with compile or run-time enforcement mechanisms to catch those usage bugs early.

### <a name="Rconc-races"></a>CP.2: Avoid data races

##### Reason

Unless you do, nothing is guaranteed to work and subtle errors will persist.

##### Note

In a nutshell, if two threads can access the same object concurrently (without synchronization), and at least one is a writer (performing a non-`const` operation), you have a data race.
For further information of how to use synchronization well to eliminate data races, please consult a good book about concurrency.

##### Example, bad

There are many examples of data races that exist, some of which are running in
production software at this very moment. One very simple example:

    int get_id() {
      static int id = 1;
      return id++;
    }

The increment here is an example of a data race. This can go wrong in many ways,
including:

* Thread A loads the value of `id`, the OS context switches A out for some
  period, during which other threads create hundreds of IDs. Thread A is then
  allowed to run again, and `id` is written back to that location as A's read of
  `id` plus one.
* Thread A and B load `id` and increment it simultaneously.  They both get the
  same ID.

Local static variables are a common source of data races.

##### Example, bad:

    void f(fstream&  fs, regex pat)
    {
        array<double, max> buf;
        int sz = read_vec(fs, buf, max);            // read from fs into buf
        gsl::span<double> s {buf};
        // ...
        auto h1 = async([&]{ sort(par, s); });     // spawn a task to sort
        // ...
        auto h2 = async([&]{ return find_all(buf, sz, pat); });   // span a task to find matches
        // ...
    }

Here, we have a (nasty) data race on the elements of `buf` (`sort` will both read and write).
All data races are nasty.
Here, we managed to get a data race on data on the stack.
Not all data races are as easy to spot as this one.

##### Example, bad:

    // code not controlled by a lock

    unsigned val;

    if (val < 5) {
        // ... other thread can change val here ...
        switch (val) {
        case 0: // ...
        case 1: // ...
        case 2: // ...
        case 3: // ...
        case 4: // ...
        }
    }

Now, a compiler that does not know that `val` can change will  most likely implement that `switch` using a jump table with five entries.
Then, a `val` outside the `[0..4]` range will cause a jump to an address that could be anywhere in the program, and execution would proceed there.
Really, "all bets are off" if you get a data race.
Actually, it can be worse still: by looking at the generated code you may be able to determine where the stray jump will go for a given value;
this can be a security risk.

##### Enforcement

Some is possible, do at least something.
There are commercial and open-source tools that try to address this problem,
but be aware that solutions have costs and blind spots.
Static tools often have many false positives and run-time tools often have a significant cost.
We hope for better tools.
Using multiple tools can catch more problems than a single one.
>>>>>>> a47e5a627f7b2ef31767de0251fd8958c85a16ee

There are other ways you can mitigate the chance of data races:

* Avoid global data
<<<<<<< HEAD
* Avoid
  `static`
  variables
* More use of value types on the stack \(and don’t pass pointers around too much\)
* More use of immutable data \(literals,
  `constexpr`
  , and
  `const`
  \)

### CP.3: Minimize explicit sharing of writable data {#cp3-minimize-explicit-sharing-of-writable-data}

##### Reason {#reason-261}

If you don’t share writable data, you can’t have a data race. The less sharing you do, the less chance you have to forget to synchronize access \(and get data races\). The less sharing you do, the less chance you have to wait on a lock \(so performance can improve\).

##### Example {#example-226}

```
bool validate(const vector
<
Reading
>
&
);
Graph
<
Temp_node
>
 temperature_gradiants(const vector
<
Reading
>
&
);
Image altitude_map(const vector
<
Reading
>
&
);
// ...

void process_readings(istream
&
 socket1)
{
    vector
<
Reading
>
 surface_readings;
    socket1 
>
>
 surface_readings;
    if (!socket1) throw Bad_input{};

    auto h1 = async([
&
] { if (!validate(surface_readings) throw Invalide_data{}; });
    auto h2 = async([
&
] { return temperature_gradiants(surface_readings); });
    auto h3 = async([
&
] { return altitude_map(surface_readings); });
    // ...
    auto v1 = h1.get();
    auto v2 = h2.get();
    auto v3 = h3.get();
    // ...
}

```

Without those`const`s, we would have to review every asynchronously invoked function for potential data races on`surface_readings`.

##### Note {#note-210}

Immutable data can be safely and efficiently shared. No locking is needed: You can’t have a data race on a constant.

##### Enforcement {#enforcement-241}

???

### CP.4: Think in terms of tasks, rather than threads {#cp4-think-in-terms-of-tasks-rather-than-threads}

##### Reason {#reason-262}

A`thread`is an implementation concept, a way of thinking about the machine. A task is an application notion, something you’d like to do, preferably concurrently with other tasks. Application concepts are easier to reason about.

##### Example {#example-227}

```
???

```

##### Note {#note-211}

With the exception of`async()`, the standard-library facilities are low-level, machine-oriented, threads-and-lock level. This is a necessary foundation, but we have to try to raise the level of abstraction: for productivity, for reliability, and for performance. This is a potent argument for using higher level, more applications-oriented libraries \(if possibly, built on top of standard-library facilities\).

##### Enforcement {#enforcement-242}

???

### CP.8: Don’t try to use`volatile`for synchronization {#cp8-dont-try-to-use-volatile-for-synchronization}

##### Reason {#reason-263}

In C++, unlike some other languages,`volatile`does not provide atomicity, does not synchronize between threads, and does not prevent instruction reordering \(neither compiler nor hardware\). It simply has nothing to do with concurrency.

##### Example, bad: {#example-bad-103}

```
int free_slots = max_slots; // current source of memory for objects

Pool* use()
{
    if (int n = free_slots--) return 
&
pool[n];
}

```

Here we have a problem: This is perfectly good code in a single-threaded program, but have two treads execute this and there is a race condition on`free_slots`so that two threads might get the same value and`free_slots`. That’s \(obviously\) a bad data race, so people trained in other languages may try to fix it like this:

```
volatile int free_slots = max_slots; // current source of memory for objects

Pool* use()
{
    if (int n = free_slots--) return 
&
pool[n];
}

```

This has no effect on synchronization: The data race is still there!

The C++ mechanism for this is`atomic`types:

```
atomic
<
int
>
 free_slots = max_slots; // current source of memory for objects

Pool* use()
{
    if (int n = free_slots--) return 
&
pool[n];
}

```

Now the`--`operation is atomic, rather than a read-increment-write sequence where another thread might get in-between the individual operations.

##### Alternative {#alternative-8}

Use`atomic`types where you might have used`volatile`in some other language. Use a`mutex`for more complicated examples.

##### See also {#see-also-2}

[\(rare\) proper uses of`volatile`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-volatile2)

### CP.9: Whenever feasible use tools to validate your concurrent code {#cp9-whenever-feasible-use-tools-to-validate-your-concurrent-code}

Experience shows that concurrent code is exceptionally hard to get right and that compile-time checking, run-time checks, and testing are less effective at finding concurrency errors than they are at finding errors in sequential code. Subtle concurrency errors can have dramatically bad effects, including memory corruption and deadlocks.

##### Example {#example-228}

```
???

```

##### Note {#note-212}

Thread safety is challenging, often getting the better of experienced programmers: tooling is an important strategy to mitigate those risks. There are many tools “out there”, both commercial and open-source tools, both research and production tools. Unfortunately people’s needs and constraints differ so dramatically that we cannot make specific recommendations, but we can mention:

* Static enforcement tools: both[clang](http://clang.llvm.org/docs/ThreadSafetyAnalysis.html)and some older versions of[GCC](https://gcc.gnu.org/wiki/ThreadSafetyAnnotation)have some support for static annotation of thread safety properties. Consistent use of this technique turns many classes of thread-safety errors into compile-time errors. The annotations are generally local \(marking a particular member variable as guarded by a particular mutex\), and are usually easy to learn. However, as with many static tools, it can often present false negatives; cases that should have been caught but were allowed.

* dynamic enforcement tools: Clang’s[Thread Sanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html)\(aka TSAN\) is a powerful example of dynamic tools: it changes the build and execution of your program to add bookkeeping on memory access, absolutely identifying data races in a given execution of your binary. The cost for this is both memory \(5-10x in most cases\) and CPU slowdown \(2-20x\). Dynamic tools like this are best when applied to integration tests, canary pushes, or unittests that operate on multiple threads. Workload matters: When TSAN identifies a problem, it is effectively always an actual data race, but it can only identify races seen in a given execution.

##### Enforcement {#enforcement-243}

It is up to an application builder to choose which support tools are valuable for a particular applications.

## CP.con: Concurrency {#cpcon-concurrency}

This section focuses on relatively ad-hoc uses of multiple threads communicating through shared data.

* For parallel algorithms, see
  [parallelism](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-par)
* For inter-task communication without explicit sharing, see
  [messaging](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-mess)
* For vector parallel code, see
  [vectorization](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-vec)
* For lock-free programming, see
  [lock free](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-free)

Concurrency rule summary:

* [CP.20: Use RAII, never plain`lock()`/`unlock()`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-raii)
* [CP.21: Use`std::lock()`or`std::scoped_lock`to acquire multiple`mutex`es](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-lock)
* [CP.22: Never call unknown code while holding a lock \(e.g., a callback\)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-unknown)
* [CP.23: Think of a joining`thread`as a scoped container](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-join)
* [CP.24: Think of a detached`thread`as a global container](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-detach)
* [CP.25: Prefer`gsl::raii_thread`over`std::thread`unless you plan to`detach()`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-raii_thread)
* [CP.26: Prefer`gsl::detached_thread`over`std::thread`if you plan to`detach()`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-detached_thread)
* [CP.27: Use plain`std::thread`for`thread`s that detach based on a run-time condition \(only\)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-thread)
* [CP.28: Remember to join scoped`thread`s that are not`detach()`ed](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-join-undetached)
* [CP.30: Do not pass pointers to local variables to non-`raii_thread`s](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-pass)
* [CP.31: Pass small amounts of data between threads by value, rather than by reference or pointer](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-data-by-value)
* [CP.32: To share ownership between unrelated`thread`s use`shared_ptr`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-shared)
* [CP.40: Minimize context switching](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-switch)
* [CP.41: Minimize thread creation and destruction](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-create)
* [CP.42: Don’t`wait`without a condition](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-wait)
* [CP.43: Minimize time spent in a critical section](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-time)
* [CP.44: Remember to name your`lock_guard`s and`unique_lock`s](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-name)
* [CP.50: Define a`mutex`together with the data it protects](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-mutex)
* ??? when to use a spinlock
* ??? when to use
  `try_lock()`
* ??? when to prefer
  `lock_guard`
  over
  `unique_lock`
* ??? Time multiplexing
* ??? when/how to use
  `new thread`

### CP.20: Use RAII, never plain`lock()`/`unlock()` {#cp20-use-raii-never-plain-lockunlock}

##### Reason {#reason-264}

Avoids nasty errors from unreleased locks.

##### Example, bad {#example-bad-104}

```
mutex mtx;

void do_stuff()
{
    mtx.lock();
    // ... do stuff ...
    mtx.unlock();
}

```

Sooner or later, someone will forget the`mtx.unlock()`, place a`return`in the`... do stuff ...`, throw an exception, or something.

```
mutex mtx;

void do_stuff()
{
    unique_lock
<
mutex
>
 lck {mtx};
    // ... do stuff ...
}

```

##### Enforcement {#enforcement-244}

Flag calls of member`lock()`and`unlock()`. ???

### CP.21: Use`std::lock()`or`std::scoped_lock`to acquire multiple`mutex`es {#cp21-use-stdlock-or-stdscoped_lock-to-acquire-multiple-mutexes}

##### Reason {#reason-265}

To avoid deadlocks on multiple`mutex`es.

##### Example {#example-229}

This is asking for deadlock:

```
// thread 1
lock_guard
<
mutex
>
 lck1(m1);
lock_guard
<
mutex
>
 lck2(m2);

// thread 2
lock_guard
<
mutex
>
 lck2(m2);
lock_guard
<
mutex
>
 lck1(m1);

```

Instead, use`lock()`:

```
// thread 1
lock(lck1, lck2);
lock_guard
<
mutex
>
 lck1(m1, adopt_lock);
lock_guard
<
mutex
>
 lck2(m2, adopt_lock);

// thread 2
lock(lck2, lck1);
lock_guard
<
mutex
>
 lck2(m2, adopt_lock);
lock_guard
<
mutex
>
 lck1(m1, adopt_lock);

```

or \(better, but C++17 only\):

```
// thread 1
scoped_lock
<
mutex, mutex
>
 lck1(m1, m2);

// thread 2
scoped_lock
<
mutex, mutex
>
 lck2(m2, m1);

```

Here, the writers of`thread1`and`thread2`are still not agreeing on the order of the`mutex`es, but order no longer matters.

##### Note {#note-213}

In real code,`mutex`es are rarely named to conveniently remind the programmer of an intended relation and intended order of acquisition. In real code,`mutex`es are not always conveniently acquired on consecutive lines.

In C++17 it’s possible to write plain

```
lock_guard lck1(m1, adopt_lock);

```

and have the`mutex`type deduced.

##### Enforcement {#enforcement-245}

Detect the acquisition of multiple`mutex`es. This is undecidable in general, but catching common simple examples \(like the one above\) is easy.

### CP.22: Never call unknown code while holding a lock \(e.g., a callback\) {#cp22-never-call-unknown-code-while-holding-a-lock-eg-a-callback}

##### Reason {#reason-266}

If you don’t know what a piece of code does, you are risking deadlock.

##### Example {#example-230}

```
void do_this(Foo* p)
{
    lock_guard
<
mutex
>
 lck {my_mutex};
    // ... do something ...
    p-
>
act(my_data);
    // ...
}

```

If you don’t know what`Foo::act`does \(maybe it is a virtual function invoking a derived class member of a class not yet written\), it may call`do_this`\(recursively\) and cause a deadlock on`my_mutex`. Maybe it will lock on a different mutex and not return in a reasonable time, causing delays to any code calling`do_this`.

##### Example {#example-231}

A common example of the “calling unknown code” problem is a call to a function that tries to gain locked access to the same object. Such problem can often be solved by using a`recursive_mutex`. For example:

```
recursive_mutex my_mutex;

template
<
typename Action
>

void do_something(Action f)
{
    unique_lock
<
recursive_mutex
>
 lck {my_mutex};
    // ... do something ...
    f(this);    // f will do something to *this
    // ...
}

```

If, as it is likely,`f()`invokes operations on`*this`, we must make sure that the object’s invariant holds before the call.

##### Enforcement {#enforcement-246}

* Flag calling a virtual function with a non-recursive
  `mutex`
  held
* Flag calling a callback with a non-recursive
  `mutex`
  held

### CP.23: Think of a joining`thread`as a scoped container {#cp23-think-of-a-joining-thread-as-a-scoped-container}

##### Reason {#reason-267}

To maintain pointer safety and avoid leaks, we need to consider what pointers are used by a`thread`. If a`thread`joins, we can safely pass pointers to objects in the scope of the`thread`and its enclosing scopes.

##### Example {#example-232}

```
void f(int * p)
{
    // ...
    *p = 99;
    // ...
}
int glob = 33;

void some_fct(int* p)
{
    int x = 77;
    raii_thread t0(f, 
&
x);           // OK
    raii_thread t1(f, p);            // OK
    raii_thread t2(f, 
&
glob);        // OK
    auto q = make_unique
<
int
>
(99);
    raii_thread t3(f, q.get());      // OK
    // ...
}

```

An`raii_thread`is a`std::thread`with a destructor that joined and cannot be`detached()`. By “OK” we mean that the object will be in scope \(“live”\) for as long as a`thread`can use the pointer to it. The fact that`thread`s run concurrently doesn’t affect the lifetime or ownership issues here; these`thread`s can be seen as just a function object called from`some_fct`.

##### Enforcement {#enforcement-247}

Ensure that`raii_thread`s don’t`detach()`. After that, the usual lifetime and ownership \(for local objects\) enforcement applies.

### CP.24: Think of a detached`thread`as a global container {#cp24-think-of-a-detached-thread-as-a-global-container}

##### Reason {#reason-268}

To maintain pointer safety and avoid leaks, we need to consider what pointers are used by a`thread`. If a`thread`is detached, we can safely pass pointers to static and free store objects \(only\).

##### Example {#example-233}

```
void f(int * p)
{
    // ...
    *p = 99;
    // ...
}

int glob = 33;

void some_fct(int* p)
{
    int x = 77;
    std::thread t0(f, 
&
x);           // bad
    std::thread t1(f, p);            // bad
    std::thread t2(f, 
&
glob);        // OK
    auto q = make_unique
<
int
>
(99);
    std::thread t3(f, q.get());      // bad
    // ...
    t0.detach();
    t1.detach();
    t2.detach();
    t3.detach();
    // ...
}

```

By “OK” we mean that the object will be in scope \(“live”\) for as long as a`thread`can use the pointers to it. By “bad” we mean that a`thread`may use a pointer after the pointed-to object is destroyed. The fact that`thread`s run concurrently doesn’t affect the lifetime or ownership issues here; these`thread`s can be seen as just a function object called from`some_fct`.

##### Note {#note-214}

Even objects with static storage duration can be problematic if used from detached threads: if the thread continues until the end of the program, it might be running concurrently with the destruction of objects with static storage duration, and thus accesses to such objects might race.

##### Enforcement {#enforcement-248}

In general, it is undecidable whether a`detach()`is executed for a`thread`, but simple common cases are easily detected. If we cannot prove that a`thread`does not`detach()`, we must assume that it does and that it outlives the scope in which it was constructed; After that, the usual lifetime and ownership \(for global objects\) enforcement applies.

### CP.25: Prefer`gsl::raii_thread`over`std::thread`unless you plan to`detach()` {#cp25-prefer-gslraii_thread-over-stdthread-unless-you-plan-to-detach}

##### Reason {#reason-269}

An`raii_thread`is a thread that joins at the end of its scope.

Detached threads are hard to monitor.

??? Place all “immortal threads” on the free store rather than`detach()`?

##### Example {#example-234}

```
???

```

##### Enforcement {#enforcement-249}

???

### CP.26: Prefer`gsl::detached_thread`over`std::thread`if you plan to`detach()` {#cp26-prefer-gsldetached_thread-over-stdthread-if-you-plan-to-detach}

#####   {#reason-270}



=======
* Avoid `static` variables
* More use of value types on the stack (and don't pass pointers around too much)
* More use of immutable data (literals, `constexpr`, and `const`)

### <a name="Rconc-data"></a>CP.3: Minimize explicit sharing of writable data

##### Reason

If you don't share writable data, you can't have a data race.
The less sharing you do, the less chance you have to forget to synchronize access (and get data races).
The less sharing you do, the less chance you have to wait on a lock (so performance can improve).

##### Example

    bool validate(const vector<Reading>&);
    Graph<Temp_node> temperature_gradiants(const vector<Reading>&);
    Image altitude_map(const vector<Reading>&);
    // ...

    void process_readings(istream& socket1)
    {
        vector<Reading> surface_readings;
        socket1 >> surface_readings;
        if (!socket1) throw Bad_input{};

        auto h1 = async([&] { if (!validate(surface_readings) throw Invalide_data{}; });
        auto h2 = async([&] { return temperature_gradiants(surface_readings); });
        auto h3 = async([&] { return altitude_map(surface_readings); });
        // ...
        auto v1 = h1.get();
        auto v2 = h2.get();
        auto v3 = h3.get();
        // ...
    }

Without those `const`s, we would have to review every asynchronously invoked function for potential data races on `surface_readings`.

##### Note

Immutable data can be safely and efficiently shared.
No locking is needed: You can't have a data race on a constant.

##### Enforcement

???


### <a name="Rconc-task"></a>CP.4: Think in terms of tasks, rather than threads

##### Reason

A `thread` is an implementation concept, a way of thinking about the machine.
A task is an application notion, something you'd like to do, preferably concurrently with other tasks.
Application concepts are easier to reason about.

##### Example

    ???

##### Note

With the exception of `async()`, the standard-library facilities are low-level, machine-oriented, threads-and-lock level.
This is a necessary foundation, but we have to try to raise the level of abstraction: for productivity, for reliability, and for performance.
This is a potent argument for using higher level, more applications-oriented libraries (if possibly, built on top of standard-library facilities).

##### Enforcement

???

### <a name="Rconc-volatile"></a>CP.8: Don't try to use `volatile` for synchronization

##### Reason

In C++, unlike some other languages, `volatile` does not provide atomicity, does not synchronize between threads,
and does not prevent instruction reordering (neither compiler nor hardware).
It simply has nothing to do with concurrency.

##### Example, bad:

    int free_slots = max_slots; // current source of memory for objects

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }

Here we have a problem:
This is perfectly good code in a single-threaded program, but have two treads execute this and
there is a race condition on `free_slots` so that two threads might get the same value and `free_slots`.
That's (obviously) a bad data race, so people trained in other languages may try to fix it like this:

    volatile int free_slots = max_slots; // current source of memory for objects

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }

This has no effect on synchronization: The data race is still there!

The C++ mechanism for this is `atomic` types:

    atomic<int> free_slots = max_slots; // current source of memory for objects

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }

Now the `--` operation is atomic,
rather than a read-increment-write sequence where another thread might get in-between the individual operations.

##### Alternative

Use `atomic` types where you might have used `volatile` in some other language.
Use a `mutex` for more complicated examples.

##### See also

[(rare) proper uses of `volatile`](#Rconc-volatile2)

### <a name="Rconc-tools"></a>CP.9: Whenever feasible use tools to validate your concurrent code

Experience shows that concurrent code is exceptionally hard to get right
and that compile-time checking, run-time checks, and testing are less effective at finding concurrency errors
than they are at finding errors in sequential code.
Subtle concurrency errors can have dramatically bad effects, including memory corruption and deadlocks.

##### Example

    ???

##### Note

Thread safety is challenging, often getting the better of experienced programmers: tooling is an important strategy to mitigate those risks.
There are many tools "out there", both commercial and open-source tools, both research and production tools.
Unfortunately people's needs and constraints differ so dramatically that we cannot make specific recommendations,
but we can mention:

* Static enforcement tools: both [clang](http://clang.llvm.org/docs/ThreadSafetyAnalysis.html)
and some older versions of [GCC](https://gcc.gnu.org/wiki/ThreadSafetyAnnotation)
have some support for static annotation of thread safety properties.
Consistent use of this technique turns many classes of thread-safety errors into compile-time errors.
The annotations are generally local (marking a particular member variable as guarded by a particular mutex),
and are usually easy to learn. However, as with many static tools, it can often present false negatives;
cases that should have been caught but were allowed.

* dynamic enforcement tools: Clang's [Thread Sanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html) (aka TSAN)
is a powerful example of dynamic tools: it changes the build and execution of your program to add bookkeeping on memory access,
absolutely identifying data races in a given execution of your binary.
The cost for this is both memory (5-10x in most cases) and CPU slowdown (2-20x).
Dynamic tools like this are best when applied to integration tests, canary pushes, or unittests that operate on multiple threads.
Workload matters: When TSAN identifies a problem, it is effectively always an actual data race,
but it can only identify races seen in a given execution.

##### Enforcement

It is up to an application builder to choose which support tools are valuable for a particular applications.

## <a name="SScp-con"></a>CP.con: Concurrency

This section focuses on relatively ad-hoc uses of multiple threads communicating through shared data.

* For parallel algorithms, see [parallelism](#SScp-par)
* For inter-task communication without explicit sharing, see [messaging](#SScp-mess)
* For vector parallel code, see [vectorization](#SScp-vec)
* For lock-free programming, see [lock free](#SScp-free)

Concurrency rule summary:

* [CP.20: Use RAII, never plain `lock()`/`unlock()`](#Rconc-raii)
* [CP.21: Use `std::lock()` or `std::scoped_lock` to acquire multiple `mutex`es](#Rconc-lock)
* [CP.22: Never call unknown code while holding a lock (e.g., a callback)](#Rconc-unknown)
* [CP.23: Think of a joining `thread` as a scoped container](#Rconc-join)
* [CP.24: Think of a detached `thread` as a global container](#Rconc-detach)
* [CP.25: Prefer `gsl::raii_thread` over `std::thread` unless you plan to `detach()`](#Rconc-raii_thread)
* [CP.26: Prefer `gsl::detached_thread` over `std::thread` if you plan to `detach()`](#Rconc-detached_thread)
* [CP.27: Use plain `std::thread` for `thread`s that detach based on a run-time condition (only)](#Rconc-thread)
* [CP.28: Remember to join scoped `thread`s that are not `detach()`ed](#Rconc-join-undetached)
* [CP.30: Do not pass pointers to local variables to non-`raii_thread`s](#Rconc-pass)
* [CP.31: Pass small amounts of data between threads by value, rather than by reference or pointer](#Rconc-data-by-value)
* [CP.32: To share ownership between unrelated `thread`s use `shared_ptr`](#Rconc-shared)
* [CP.40: Minimize context switching](#Rconc-switch)
* [CP.41: Minimize thread creation and destruction](#Rconc-create)
* [CP.42: Don't `wait` without a condition](#Rconc-wait)
* [CP.43: Minimize time spent in a critical section](#Rconc-time)
* [CP.44: Remember to name your `lock_guard`s and `unique_lock`s](#Rconc-name)
* [CP.50: Define a `mutex` together with the data it protects](#Rconc-mutex)
* ??? when to use a spinlock
* ??? when to use `try_lock()`
* ??? when to prefer `lock_guard` over `unique_lock`
* ??? Time multiplexing
* ??? when/how to use `new thread`

### <a name="Rconc-raii"></a>CP.20: Use RAII, never plain `lock()`/`unlock()`

##### Reason

Avoids nasty errors from unreleased locks.

##### Example, bad

    mutex mtx;

    void do_stuff()
    {
        mtx.lock();
        // ... do stuff ...
        mtx.unlock();
    }

Sooner or later, someone will forget the `mtx.unlock()`, place a `return` in the `... do stuff ...`, throw an exception, or something.

    mutex mtx;

    void do_stuff()
    {
        unique_lock<mutex> lck {mtx};
        // ... do stuff ...
    }

##### Enforcement

Flag calls of member `lock()` and `unlock()`.  ???


### <a name="Rconc-lock"></a>CP.21: Use `std::lock()` or `std::scoped_lock` to acquire multiple `mutex`es

##### Reason

To avoid deadlocks on multiple `mutex`es.

##### Example

This is asking for deadlock:

    // thread 1
    lock_guard<mutex> lck1(m1);
    lock_guard<mutex> lck2(m2);

    // thread 2
    lock_guard<mutex> lck2(m2);
    lock_guard<mutex> lck1(m1);

Instead, use `lock()`:

    // thread 1
    lock(lck1, lck2);
    lock_guard<mutex> lck1(m1, adopt_lock);
    lock_guard<mutex> lck2(m2, adopt_lock);

    // thread 2
    lock(lck2, lck1);
    lock_guard<mutex> lck2(m2, adopt_lock);
    lock_guard<mutex> lck1(m1, adopt_lock);

or (better, but C++17 only):

    // thread 1
    scoped_lock<mutex, mutex> lck1(m1, m2);

    // thread 2
    scoped_lock<mutex, mutex> lck2(m2, m1);

Here, the writers of `thread1` and `thread2` are still not agreeing on the order of the `mutex`es, but order no longer matters.

##### Note

In real code, `mutex`es are rarely named to conveniently remind the programmer of an intended relation and intended order of acquisition.
In real code, `mutex`es are not always conveniently acquired on consecutive lines.

In C++17 it's possible to write plain

    lock_guard lck1(m1, adopt_lock);

and have the `mutex` type deduced.

##### Enforcement

Detect the acquisition of multiple `mutex`es.
This is undecidable in general, but catching common simple examples (like the one above) is easy.


### <a name="Rconc-unknown"></a>CP.22: Never call unknown code while holding a lock (e.g., a callback)

##### Reason

If you don't know what a piece of code does, you are risking deadlock.

##### Example

    void do_this(Foo* p)
    {
        lock_guard<mutex> lck {my_mutex};
        // ... do something ...
        p->act(my_data);
        // ...
    }

If you don't know what `Foo::act` does (maybe it is a virtual function invoking a derived class member of a class not yet written),
it may call `do_this` (recursively) and cause a deadlock on `my_mutex`.
Maybe it will lock on a different mutex and not return in a reasonable time, causing delays to any code calling `do_this`.

##### Example

A common example of the "calling unknown code" problem is a call to a function that tries to gain locked access to the same object.
Such problem can often be solved by using a `recursive_mutex`. For example:

    recursive_mutex my_mutex;

    template<typename Action>
    void do_something(Action f)
    {
        unique_lock<recursive_mutex> lck {my_mutex};
        // ... do something ...
        f(this);    // f will do something to *this
        // ...
    }

If, as it is likely, `f()` invokes operations on `*this`, we must make sure that the object's invariant holds before the call.

##### Enforcement

* Flag calling a virtual function with a non-recursive `mutex` held
* Flag calling a callback with a non-recursive `mutex` held


### <a name="Rconc-join"></a>CP.23: Think of a joining `thread` as a scoped container

##### Reason

To maintain pointer safety and avoid leaks, we need to consider what pointers are used by a `thread`.
If a `thread` joins, we can safely pass pointers to objects in the scope of the `thread` and its enclosing scopes.

##### Example

    void f(int * p)
    {
        // ...
        *p = 99;
        // ...
    }
    int glob = 33;

    void some_fct(int* p)
    {
        int x = 77;
        raii_thread t0(f, &x);           // OK
        raii_thread t1(f, p);            // OK
        raii_thread t2(f, &glob);        // OK
        auto q = make_unique<int>(99);
        raii_thread t3(f, q.get());      // OK
        // ...
    }

An `raii_thread` is a `std::thread` with a destructor that joined and cannot be `detached()`.
By "OK" we mean that the object will be in scope ("live") for as long as a `thread` can use the pointer to it.
The fact that `thread`s run concurrently doesn't affect the lifetime or ownership issues here;
these `thread`s can be seen as just a function object called from `some_fct`.

##### Enforcement

Ensure that `raii_thread`s don't `detach()`.
After that, the usual lifetime and ownership (for local objects) enforcement applies.


### <a name="Rconc-detach"></a>CP.24: Think of a detached `thread` as a global container

##### Reason

To maintain pointer safety and avoid leaks, we need to consider what pointers are used by a `thread`.
If a `thread` is detached, we can safely pass pointers to static and free store objects (only).

##### Example

    void f(int * p)
    {
        // ...
        *p = 99;
        // ...
    }

    int glob = 33;

    void some_fct(int* p)
    {
        int x = 77;
        std::thread t0(f, &x);           // bad
        std::thread t1(f, p);            // bad
        std::thread t2(f, &glob);        // OK
        auto q = make_unique<int>(99);
        std::thread t3(f, q.get());      // bad
        // ...
        t0.detach();
        t1.detach();
        t2.detach();
        t3.detach();
        // ...
    }

By "OK" we mean that the object will be in scope ("live") for as long as a `thread` can use the pointers to it.
By "bad" we mean that a `thread` may use a pointer after the pointed-to object is destroyed.
The fact that `thread`s run concurrently doesn't affect the lifetime or ownership issues here;
these `thread`s can be seen as just a function object called from `some_fct`.

##### Note

Even objects with static storage duration can be problematic if used from detached threads: if the
thread continues until the end of the program, it might be running concurrently with the destruction
of objects with static storage duration, and thus accesses to such objects might race.

##### Enforcement

In general, it is undecidable whether a `detach()` is executed for a `thread`, but simple common cases are easily detected.
If we cannot prove that a `thread` does not `detach()`, we must assume that it does and that it outlives the scope in which it was constructed;
After that, the usual lifetime and ownership (for global objects) enforcement applies.


### <a name="Rconc-raii_thread"></a>CP.25: Prefer `gsl::raii_thread` over `std::thread` unless you plan to `detach()`

##### Reason

An `raii_thread` is a thread that joins at the end of its scope.

Detached threads are hard to monitor.

??? Place all "immortal threads" on the free store rather than `detach()`?

##### Example

    ???

##### Enforcement

???

### <a name="Rconc-detached_thread"></a>CP.26: Prefer `gsl::detached_thread` over `std::thread` if you plan to `detach()`

##### Reason

Often, the need to `detach` is inherent in the `thread`s task.
Documenting that aids comprehension and helps static analysis.

##### Example

    void heartbeat();

    void use()
    {
        gsl::detached_thread t1(heartbeat);    // obviously need not be joined
        std::thread t2(heartbeat);             // do we need to join? (read the code for heartbeat())
        // ...
    }

Flag unconditional `detach` on a plain `thread`


### <a name="Rconc-thread"></a>CP.27: Use plain `std::thread` for `thread`s that detach based on a run-time condition (only)

##### Reason

`thread`s that are supposed to unconditionally `join` or unconditionally `detach` can be clearly identified as such.
The plain `thread`s should be assumed to use the full generality of `std::thread`.

##### Example

    void tricky(thread* t, int n)
    {
        // ...
        if (is_odd(n))
            t->detach();
        // ...
    }

    void use(int n)
    {
        thread t { tricky, this, n };
        // ...
        // ... should I join here? ...
    }

##### Enforcement

???



### <a name="Rconc-join-undetached"></a>CP.28: Remember to join scoped `thread`s that are not `detach()`ed

##### Reason

A `thread` that has not been `detach()`ed when it is destroyed terminates the program.

##### Example, bad

    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() executes in separate thread
        std::thread t2{F()};    // F()() executes in separate thread
    }  // spot the bugs

##### Example

    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() executes in separate thread
        std::thread t2{F()};    // F()() executes in separate thread

        t1.join();
        t2.join();
    }  // one bad bug left

??? Is `cout` synchronized?

##### Enforcement

* Flag `join`s for `raii_thread`s ???
* Flag `detach`s for `detached_thread`s


### <a name="RRconc-pass"></a>CP.30: Do not pass pointers to local variables to non-`raii_thread`s

##### Reason

In general, you cannot know whether a non-`raii_thread` will outlive the scope of the variables, so that those pointers will become invalid.

##### Example, bad

    void use()
    {
        int x = 7;
        thread t0 { f, ref(x) };
        // ...
        t0.detach();
    }

The `detach` may not be so easy to spot.
Use a `raii_thread` or don't pass the pointer.

##### Example, bad

    ??? put pointer to a local on a queue that is read by a longer-lived thread ???

##### Enforcement

Flag pointers to locals passed in the constructor of a plain `thread`.


### <a name="Rconc-data-by-value"></a>CP.31: Pass small amounts of data between threads by value, rather than by reference or pointer

##### Reason

Copying a small amount of data is cheaper to copy and access than to share it using some locking mechanism.
Copying naturally gives unique ownership (simplifies code) and eliminates the possibility of data races.

##### Note

Defining "small amount" precisely is impossible.

##### Example

    string modify1(string);
    void modify2(shared_ptr<string>);

    void fct(string& s)
    {
        auto res = async(modify1, s);
        async(modify2, &s);
    }

The call of `modify1` involves copying two `string` values; the call of `modify2` does not.
On the other hand, the implementation of `modify1` is exactly as we would have written it for single-threaded code,
whereas the implementation of `modify2` will need some form of locking to avoid data races.
If the string is short (say 10 characters), the call of `modify1` can be surprisingly fast;
essentially all the cost is in the `thread` switch. If the string is long (say 1,000,000 characters), copying it twice
is probably not a good idea.

Note that this argument has nothing to do with `sync` as such. It applies equally to considerations about whether to use
message passing or shared memory.

##### Enforcement

???


### <a name="Rconc-shared"></a>[CP.32: To share ownership between unrelated `thread`s use `shared_ptr`

##### Reason

If threads are unrelated (that is, not known to be in the same scope or one within the lifetime of the other)
and they need to share free store memory that needs to be deleted, a `shared_ptr` (or equivalent) is the only
safe way to ensure proper deletion.

##### Example

    ???

##### Note

* A static object (e.g. a global) can be shared because it is not owned in the sense that some thread is responsible for it's deletion.
* An object on free store that is never to be deleted can be shared.
* An object owned by one thread can be safely shared with another as long as that second thread doesn't outlive the owner.

##### Enforcement

???


### <a name="Rconc-switch"></a>CP.40: Minimize context switching

##### Reason

Context switches are expensive.

##### Example

    ???

##### Enforcement

???


### <a name="Rconc-create"></a>CP.41: Minimize thread creation and destruction

##### Reason

Thread creation is expensive.

##### Example

    void worker(Message m)
    {
        // process
    }

    void master(istream& is)
    {
        for (Message m; is >> m; )
            run_list.push_back(new thread(worker, m));
    }

This spawns a `thread` per message, and the `run_list` is presumably managed to destroy those tasks once they are finished.

Instead, we could have a set of pre-created worker threads processing the messages

    Sync_queue<Message> work;

    void master(istream& is)
    {
        for (Message m; is >> m; )
            work.put(m);
    }

    void worker()
    {
        for (Message m; m = work.get(); ) {
            // process
        }
    }

    void workers()  // set up worker threads (specifically 4 worker threads)
    {
        raii_thread w1 {worker};
        raii_thread w2 {worker};
        raii_thread w3 {worker};
        raii_thread w4 {worker};
    }

##### Note

If your system has a good thread pool, use it.
If your system has a good message queue, use it.

##### Enforcement

???


### <a name="Rconc-wait"></a>CP.42: Don't `wait` without a condition

##### Reason

A `wait` without a condition can miss a wakeup or wake up simply to find that there is no work to do.

##### Example, bad

    std::condition_variable cv;
    std::mutex mx;

    void thread1()
    {
        while (true) {
            // do some work ...
            std::unique_lock<std::mutex> lock(mx);
            cv.notify_one();    // wake other thread
        }
    }

    void thread2()
    {
        while (true) {
            std::unique_lock<std::mutex> lock(mx);
            cv.wait(lock);    // might block forever
            // do work ...
        }
    }

Here, if some other `thread` consumes `thread1`'s notification, `thread2` can wait forever.

##### Example

    template<typename T>
    class Sync_queue {
    public:
        void put(const T& val);
        void put(T&& val);
        void get(T& val);
    private:
        mutex mtx;
        condition_variable cond;    // this controls access
        list<T> q;
    };

    template<typename T>
    void Sync_queue<T>::put(const T& val)
    {
        lock_guard<mutex> lck(mtx);
        q.push_back(val);
        cond.notify_one();
    }

    template<typename T>
    void Sync_queue<T>::get(T& val)
    {
        unique_lock<mutex> lck(mtx);
        cond.wait(lck, [this]{ return !q.empty(); });    // prevent spurious wakeup
        val = q.front();
        q.pop_front();
    }

Now if the queue is empty when a thread executing `get()` wakes up (e.g., because another thread has gotten to `get()` before it),
it will immediately go back to sleep, waiting.

##### Enforcement

Flag all `wait`s without conditions.


### <a name="Rconc-time"></a>CP.43: Minimize time spent in a critical section

##### Reason

The less time is spent with a `mutex` taken, the less chance that another `thread` has to wait,
and `thread` suspension and resumption are expensive.

##### Example

    void do_something() // bad
    {
        unique_lock<mutex> lck(my_lock);
        do0();  // preparation: does not need lock
        do1();  // transaction: needs locking
        do2();  // cleanup: does not need locking
    }

Here, we are holding the lock for longer than necessary:
We should not have taken the lock before we needed it and should have released it again before starting the cleanup.
We could rewrite this to

    void do_something() // bad
    {
        do0();  // preparation: does not need lock
        my_lock.lock();
        do1();  // transaction: needs locking
        my_lock.unlock();
        do2();  // cleanup: does not need locking
    }

But that compromises safety and violates the [use RAII](#Rconc-raii) rule.
Instead, add a block for the critical section:

    void do_something() // OK
    {
        do0();  // preparation: does not need lock
        {
            unique_lock<mutex> lck(my_lock);
            do1();  // transaction: needs locking
        }
        do2();  // cleanup: does not need locking
    }

##### Enforcement

Impossible in general.
Flag "naked" `lock()` and `unlock()`.


### <a name="Rconc-name"></a>CP.44: Remember to name your `lock_guard`s and `unique_lock`s

##### Reason

An unnamed local objects is a temporary that immediately goes out of scope.

##### Example

    unique_lock<mutex>(m1);
    lock_guard<mutex> {m2};
    lock(m1, m2);

This looks innocent enough, but it isn't.

##### Enforcement

Flag all unnamed `lock_guard`s and `unique_lock`s.



### <a name="Rconc-mutex"></a>P.50: Define a `mutex` together with the data it guards. Use `synchronized_value<T>` where possible

##### Reason

It should be obvious to a reader that the data is to be guarded and how. This decreases the chance of the wrong mutex being locked, or the mutex not being locked. 

Using a `synchronized_value<T>` ensures that the data has a mutex, and the right mutex is locked when the data is accessed.
See the [WG21 proposal](http://wg21.link/p0290)) to add `synchronized_value` to a future TS or revision of the C++ standard.

##### Example

    struct Record {
        std::mutex m;   // take this mutex before accessing other members
        // ...
    };

    class MyClass {
        struct DataRecord {
           // ...
        };
        synchronized_value<DataRecord> data; // Protect the data with a mutex
    };

##### Enforcement

??? Possible?


## <a name="SScp-par"></a>CP.par: Parallelism

By "parallelism" we refer to performing a task (more or less) simultaneously ("in parallel with") on many data items.

Parallelism rule summary:

* ???
* ???
* Where appropriate, prefer the standard-library parallel algorithms
* Use algorithms that are designed for parallelism, not algorithms with unnecessary dependency on linear evaluation



## <a name="SScp-mess"></a>CP.mess: Message passing

The standard-library facilities are quite low level, focused on the needs of close-to the hardware critical programming using `thread`s, `mutex`es, `atomic` types, etc.
Most people shouldn't work at this level: it's error-prone and development is slow.
If possible, use a higher level facility: messaging libraries, parallel algorithms, and vectorization.
This section looks at passing messages so that a programmer doesn't have to do explicit synchronization.

Message passing rules summary:

* [CP.60: Use a `future` to return a value from a concurrent task](#Rconc-future)
* [CP.61: Use a `async()` to spawn a concurrent task](#Rconc-async)
* message queues
* messaging libraries

???? should there be a "use X rather than `std::async`" where X is something that would use a better specified thread pool?

??? Is `std::async` worth using in light of future (and even existing, as libraries) parallelism facilities? What should the guidelines recommend if someone wants to parallelize, e.g., `std::accumulate` (with the additional precondition of commutativity), or merge sort?


### <a name="Rconc-future"></a>CP.60: Use a `future` to return a value from a concurrent task

##### Reason

A `future` preserves the usual function call return semantics for asynchronous tasks.
The is no explicit locking and both correct (value) return and error (exception) return are handled simply.

##### Example

    ???

##### Note

???

##### Enforcement

???

### <a name="Rconc-async"></a>CP.61: Use a `async()` to spawn a concurrent task

##### Reason

A `future` preserves the usual function call return semantics for asynchronous tasks.
The is no explicit locking and both correct (value) return and error (exception) return are handled simply.

##### Example

    ???

##### Note

Unfortunately, `async()` is not perfect.
For example, there is no guarantee that a thread pool is used to minimize thread construction.
In fact, most current `async()` implementations don't.
However, `async()` is simple and logically correct so until something better comes along
and unless you really need to optimize for many asynchronous tasks, stick with `async()`.

##### Enforcement

???


## <a name="SScp-vec"></a>CP.vec: Vectorization

Vectorization is a technique for executing a number of tasks concurrently without introducing explicit synchronization.
An operation is simply applied to elements of a data structure (a vector, an array, etc.) in parallel.
Vectorization has the interesting property of often requiring no non-local changes to a program.
However, vectorization works best with simple data structures and with algorithms specifically crafted to enable it.

Vectorization rule summary:

* ???
* ???

## <a name="SScp-free"></a>CP.free: Lock-free programming

Synchronization using `mutex`es and `condition_variable`s can be relatively expensive.
Furthermore, it can lead to deadlock.
For performance and to eliminate the possibility of deadlock, we sometimes have to use the tricky low-level "lock-free" facilities
that rely on briefly gaining exclusive ("atomic") access to memory.
Lock free programming is also used to implement higher-level concurrency mechanisms, such as `thread`s and `mutex`es.

Lock-free programming rule summary:

* [CP.100: Don't use lock-free programming unless you absolutely have to](#Rconc-lockfree)
* [CP.101: Distrust your hardware/compiler combination](#Rconc-distrust)
* [CP.102: Carefully study the literature](#Rconc-literature)
* how/when to use atomics
* avoid starvation
* use a lock free data structure rather than hand-crafting specific lock-free access
* [CP.110: Do not write your own double-checked locking for initialization](#Rconc-double)
* [CP.111: Use a conventional pattern if you really need double-checked locking](#Rconc-double-pattern)
* how/when to compare and swap


### <a name="Rconc-lockfree"></a>CP.100: Don't use lock-free programming unless you absolutely have to

##### Reason

It's error-prone and requires expert level knowledge of language features, machine architecture, and data structures.

##### Example, bad

    extern atomic<Link*> head;        // the shared head of a linked list

    Link* nh = new Link(data, nullptr);    // make a link ready for insertion
    Link* h = head.load();                 // read the shared head of the list

    do {
        if (h->data <= data) break;        // if so, insert elsewhere
        nh->next = h;                      // next element is the previous head
    } while (!head.compare_exchange_weak(h, nh));    // write nh to head or to h

Spot the bug.
It would be really hard to find through testing.
Read up on the ABA problem.

##### Exception

[Atomic variables](#???) can be used simply and safely, as long as you are using the sequentially consistent memory model (memory_order_seq_cst), which is the default.

##### Note

Higher-level concurrency mechanisms, such as `thread`s and `mutex`es are implemented using lock-free programming.

**Alternative**: Use lock-free data structures implemented by others as part of some library.


### <a name="Rconc-distrust"></a>CP.101: Distrust your hardware/compiler combination

##### Reason

The low-level hardware interfaces used by lock-free programming are among the hardest to implement well and among
the areas where the most subtle portability problems occur.
If you are doing lock-free programming for performance, you need to check for regressions.

##### Note

Instruction reordering (static and dynamic) makes it hard for us to think effectively at this level (especially if you use relaxed memory models).
Experience, (semi)formal models and model checking can be useful.
Testing - often to an extreme extent - is essential.
"Don't fly too close to the sun."

##### Enforcement

Have strong rules for re-testing in place that covers any change in hardware, operating system, compiler, and libraries.


### <a name="Rconc-literature"></a>CP.102: Carefully study the literature

##### Reason

With the exception of atomics and a few use standard patterns, lock-free programming is really an expert-only topic.
Become an expert before shipping lock-free code for others to use.

##### References

* Anthony Williams: C++ concurrency in action. Manning Publications.
* Boehm, Adve, You Don't Know Jack About Shared Variables or Memory Models , Communications of the ACM, Feb 2012.
* Boehm, "Threads Basics", HPL TR 2009-259.
* Adve, Boehm, "Memory Models: A Case for Rethinking Parallel Languages and Hardware", Communications of the ACM, August 2010.
* Boehm, Adve, "Foundations of the C++ Concurrency Memory Model", PLDI 08.
* Mark Batty, Scott Owens, Susmit Sarkar, Peter Sewell, and Tjark Weber, "Mathematizing C++ Concurrency", POPL 2011.
* Damian Dechev, Peter Pirkelbauer, and Bjarne Stroustrup: Understanding and Effectively Preventing the ABA Problem in Descriptor-based Lock-free Designs. 13th IEEE Computer Society ISORC 2010 Symposium. May 2010.
* Damian Dechev and Bjarne Stroustrup: Scalable Non-blocking Concurrent Objects for Mission Critical Code. ACM OOPSLA'09. October 2009
* Damian Dechev, Peter Pirkelbauer, Nicolas Rouquette, and Bjarne Stroustrup: Semantically Enhanced Containers for Concurrent Real-Time Systems. Proc. 16th Annual IEEE International Conference and Workshop on the Engineering of Computer Based Systems (IEEE ECBS). April 2009.


### <a name="Rconc-double"></a>CP.110: Do not write your own double-checked locking for initialization

##### Reason

Since C++11, static local variables are now initialized in a thread-safe way. When combined with the RAII pattern, static local variables can replace the need for writing your own double-checked locking for initialization. std::call_once can also achieve the same purpose. Use either static local variables of C++11 or std::call_once instead of writing your own double-checked locking for initialization.

##### Example

Example with std::call_once.

    void f()
    {
        static std::once_flag my_once_flag;
        std::call_once(my_once_flag, []()
        {
            // do this only once
        });
        // ...
    }

Example with thread-safe static local variables of C++11.

    void f()
    {
        // Assuming the compiler is compliant with C++11
        static My_class my_object; // Constructor called only once
        // ...
    }

    class My_class
    {
    public:
        My_class()
        {
            // ...
        }
    };

##### Enforcement

??? Is it possible to detect the idiom?


### <a name="Rconc-double-pattern"></a>CP.111: Use a conventional pattern if you really need double-checked locking

##### Reason

Double-checked locking is easy to mess up. If you really need to write your own double-checked locking, in spite of the rules [CP.110: Do not write your own double-checked locking for initialization](#Rconc-double) and [CP.100: Don't use lock-free programming unless you absolutely have to](#Rconc-lockfree), then do it in a conventional pattern.

##### Example, bad

Even if the following example works correctly on most hardware platforms, it is not guaranteed to work by the C++ standard. The x_init.load(memory_order_relaxed) call may see a value from outside of the lock guard.

    atomic<bool> x_init;

    if (!x_init.load(memory_order_acquire)) {
        lock_guard<mutex> lck(x_mutex);
        if (!x_init.load(memory_order_relaxed)) {
            // ... initialize x ...
            x_init.store(true, memory_order_release);
        }
    }

##### Example, good

One of the conventional patterns is below.

    std::atomic<int> state;

    // If state == SOME_ACTION_NEEDED maybe an action is needed, maybe not, we need to
    // check again in a lock. However, if state != SOME_ACTION_NEEDED, then we can be
    // sure that an action is not needed. This is the basic assumption of double-checked
    // locking.

    if (state == SOME_ACTION_NEEDED)
    {
        std::lock_guard<std::mutex> lock(mutex);
        if (state == SOME_ACTION_NEEDED)
        {
            // do something
            state = NO_ACTION_NEEDED;
        }
    }

In the example above (state == SOME_ACTION_NEEDED) could be any condition. It doesn't necessarily needs to be equality comparison. For example, it could as well be (size > MIN_SIZE_TO_TAKE_ACTION).

##### Enforcement

??? Is it possible to detect the idiom?


## <a name="SScp-etc"></a>CP.etc: Etc. concurrency rules

These rules defy simple categorization:

* [CP.200: Use `volatile` only to talk to non-C++ memory](#Rconc-volatile2)
* [CP.201: ??? Signals](#Rconc-signal)

### <a name="Rconc-volatile2"></a>CP.200: Use `volatile` only to talk to non-C++ memory

##### Reason

`volatile` is used to refer to objects that are shared with "non-C++" code or hardware that does not follow the C++ memory model.

##### Example

    const volatile long clock;

This describes a register constantly updated by a clock circuit.
`clock` is `volatile` because its value will change without any action from the C++ program that uses it.
For example, reading `clock` twice will often yield two different values, so the optimizer had better not optimize away the second read in this code:

    long t1 = clock;
    // ... no use of clock here ...
    long t2 = clock;

`clock` is `const` because the program should not try to write to `clock`.

##### Note

Unless you are writing the lowest level code manipulating hardware directly, consider `volatile` an esoteric feature that is best avoided.

##### Example

Usually C++ code receives `volatile` memory that is owned Elsewhere (hardware or another language):

    int volatile* vi = get_hardware_memory_location();
        // note: we get a pointer to someone else's memory here
        // volatile says "treat this with extra respect"

Sometimes C++ code allocates the `volatile` memory and shares it with "elsewhere" (hardware or another language) by deliberately escaping a pointer:

    static volatile long vl;
    please_use_this(&vl);   // escape a reference to this to "elsewhere" (not C++)

##### Example; bad

`volatile` local variables are nearly always wrong -- how can they be shared with other languages or hardware if they're ephemeral?
The same applies almost as strongly to member variables, for the same reason.

    void f() {
        volatile int i = 0; // bad, volatile local variable
        // etc.
    }

    class My_type {
        volatile int i = 0; // suspicious, volatile member variable
        // etc.
    };

##### Note

In C++, unlike in some other languages, `volatile` has [nothing to do with synchronization](#Rconc-volatile).

##### Enforcement

* Flag `volatile T` local and member variables; almost certainly you intended to use `atomic<T>` instead.
* ???

### <a name="Rconc-signal"></a>CP.201: ??? Signals

???UNIX signal handling???. May be worth reminding how little is async-signal-safe, and how to communicate with a signal handler (best is probably "not at all")
>>>>>>> a47e5a627f7b2ef31767de0251fd8958c85a16ee
