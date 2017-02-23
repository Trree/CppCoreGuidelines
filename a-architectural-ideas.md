#
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

