# CP: Concurrency and Parallelism {#cp-concurrency-and-parallelism}

We often want our computers to do many tasks at the same time \(or at least make them appear to do them at the same time\). The reasons for doing so varies \(e.g., wanting to wait for many events using only a single processor, processing many data streams simultaneously, or utilizing many hardware facilities\) and so does the basic facilities for expressing concurrency and parallelism. Here, we articulate a few general principles and rules for using the ISO standard C++ facilities for expressing basic concurrency and parallelism.

The core machine support for concurrent and parallel programming is the thread. Threads allow you to run multiple instances of your program independently, while sharing the same memory. Concurrent programming is tricky for many reasons, most importantly that it is undefined behavior to read data in one thread after it was written by another thread, if there is no proper synchronization between those threads. Making existing single-threaded code execute concurrently can be as trivial as adding`std::async`or`std::thread`strategically, or it can necessitate a full rewrite, depending on whether the original code was written in a thread-friendly way.

The concurrency/parallelism rules in this document are designed with three goals in mind:

* To help you write code that is amenable to being used in a threaded environment
* To show clean, safe ways to use the threading primitives offered by the standard library
* To offer guidance on what to do when concurrency and parallelism aren’t giving you the performance gains you need

It is also important to note that concurrency in C++ is an unfinished story. C++11 introduced many core concurrency primitives, C++14 improved on them, and it seems that there is much interest in making the writing of concurrent programs in C++ even easier. We expect some of the library-related guidance here to change significantly over time.

This section needs a lot of work \(obviously\). Please note that we start with rules for relative non-experts. Real experts must wait a bit; contributions are welcome, but please think about the majority of programmers who are struggling to get their concurrent programs correct and performant.

Concurrency and parallelism rule summary:

* [CP.1: Assume that your code will run as part of a multi-threaded program](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-multi)
* [CP.2: Avoid data races](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-races)
* [CP.3: Minimize explicit sharing of writable data](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-data)
* [CP.4: Think in terms of tasks, rather than threads](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-task)
* [CP.8: Don’t try to use`volatile`for synchronization](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-volatile)
* [CP.9: Whenever feasible use tools to validate your concurrent code](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-tools)

See also:

* [CP.con: Concurrency](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-con)
* [CP.par: Parallelism](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-par)
* [CP.mess: Message passing](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-mess)
* [CP.vec: Vectorization](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-vec)
* [CP.free: Lock-free programming](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-free)
* [CP.etc: Etc. concurrency rules](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SScp-etc)

### CP.1: Assume that your code will run as part of a multi-threaded program {#cp1-assume-that-your-code-will-run-as-part-of-a-multi-threaded-program}

##### Reason {#reason-259}

It is hard to be certain that concurrency isn’t used now or will be sometime in the future. Code gets re-used. Libraries using threads may be used from some other part of the program. Note that this applies most urgently to library code and least urgently to stand-alone applications. However, thanks to the magic of cut-and-paste, code fragments can turn up in unexpected places.

##### Example {#example-225}

```
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

```

Although`cached_computation`works perfectly in a single-threaded environment, in a multi-threaded environment the two`static`variables result in data races and thus undefined behavior.

There are several ways that this example could be made safe for a multi-threaded environment:

* Delegate concurrency concerns upwards to the caller.
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
* Have the caller provide the memory to be used for the cache, thereby delegating both memory allocation and concurrency concerns upwards to the caller.
* Refuse to build and/or run in a multi-threaded environment.
* Provide two implementations, one which is used in single-threaded environments and another which is used in multi-threaded environments.

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

There are other ways you can mitigate the chance of data races:

* Avoid global data
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



