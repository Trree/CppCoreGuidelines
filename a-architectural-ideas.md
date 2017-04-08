# A: Architectural Ideas {#a-architectural-ideas}

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

  


