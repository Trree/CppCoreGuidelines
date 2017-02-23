# C++ Core Guidelines {#c-core-guidelines}

April 3, 2017

Editors:

* [Bjarne Stroustrup](http://www.stroustrup.com/)
* [Herb Sutter](http://herbsutter.com/)

This document is a very early draft. It is inkorrekt, incompleat, and pÂµÃoorly formatted. Had it been an open source \(code\) project, this would have been release 0.7. Copying, use, modification, and creation of derivative works from this project is licensed under an MIT-style license. Contributing to this project requires agreeing to a Contributor License. See the accompanying[LICENSE](http://isocpp.github.io/CppCoreGuidelines/LICENSE)file for details. We make this project available to “friendly users” to use, copy, modify, and derive from, hoping for constructive input.

Comments and suggestions for improvements are most welcome. We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve. When commenting, please note[the introduction](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-introduction)that outlines our aims and general approach. The list of contributors is[here](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-ack).

Problems:

* The sets of rules have not been thoroughly checked for completeness, consistency, or enforceability.
* Triple question marks \(???\) mark known missing information
* Update reference sections; many pre-C++11 sources are too old.
* For a more-or-less up-to-date to-do list see:
  [To-do: Unclassified proto-rules](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-unclassified)

You can[read an explanation of the scope and structure of this Guide](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-abstract)or just jump straight in:

* [In: Introduction](https://trree.gitbooks.io/cppcoreguidelines/content/in-introduction.html)
* [P: Philosophy](https://trree.gitbooks.io/cppcoreguidelines/content/p-philosophy.html)
* [I: Interfaces](https://trree.gitbooks.io/cppcoreguidelines/content/i-interfaces.html)
* [F: Functions](https://trree.gitbooks.io/cppcoreguidelines/content/f-functions.html)
* [C: Classes and class hierarchies](https://trree.gitbooks.io/cppcoreguidelines/content/c-classes-and-class-hierarchies.html)
* [Enum: Enumerations](https://trree.gitbooks.io/cppcoreguidelines/content/enum-enumerations.html)
* [R: Resource management](https://trree.gitbooks.io/cppcoreguidelines/content/r-resource-management.html)
* [ES: Expressions and statements](https://trree.gitbooks.io/cppcoreguidelines/content/es-expressions-and-statements.html)
* [Per: Performance](https://trree.gitbooks.io/cppcoreguidelines/content/per-performance.html)
* [CP: Concurrency](https://trree.gitbooks.io/cppcoreguidelines/content/cp-concurrency-and-parallelism.html)
* [E: Error handling](https://trree.gitbooks.io/cppcoreguidelines/content/e-error-handling.html)
* [Con: Constants and immutability](https://trree.gitbooks.io/cppcoreguidelines/content/con-constants-and-immutability.html)
* [T: Templates and generic programming](https://trree.gitbooks.io/cppcoreguidelines/content/t-templates-and-generic-programming.html)
* [CPL: C-style programming](https://trree.gitbooks.io/cppcoreguidelines/content/cpl-c-style-programming.html)
* [SF: Source files](https://trree.gitbooks.io/cppcoreguidelines/content/sf-source-files.html)
* [SL: The Standard library](https://trree.gitbooks.io/cppcoreguidelines/content/sl-the-standard-library.html)

Supporting sections:

* [A: Architectural Ideas](https://trree.gitbooks.io/cppcoreguidelines/content/a-architectural-ideas.html)
* [N: Non-Rules and myths](https://trree.gitbooks.io/cppcoreguidelines/content/nr-non-rules-and-myths.html)
* [RF: References](https://trree.gitbooks.io/cppcoreguidelines/content/rf-references.html)
* [Pro: Profiles](https://trree.gitbooks.io/cppcoreguidelines/content/pro-profiles.html)
* [GSL: Guideline support library](https://trree.gitbooks.io/cppcoreguidelines/content/gsl-guideline-support-library.html)
* [NL: Naming and layout](https://trree.gitbooks.io/cppcoreguidelines/content/nl-naming-and-layout-rules.html)
* [FAQ: Answers to frequently asked questions](https://trree.gitbooks.io/cppcoreguidelines/content/faq-answers-to-frequently-asked-questions.html)
* [Appendix A: Libraries](https://trree.gitbooks.io/cppcoreguidelines/content/appendix-a-libraries.html)
* [Appendix B: Modernizing code](https://trree.gitbooks.io/cppcoreguidelines/content/appendix-b-modernizing-code.html)
* [Appendix C: Discussion](https://trree.gitbooks.io/cppcoreguidelines/content/appendix-c-discussion.html)
* [Glossary](https://trree.gitbooks.io/cppcoreguidelines/content/glossary.html)
* [To-do: Unclassified proto-rules](https://trree.gitbooks.io/cppcoreguidelines/content/to-do-unclassified-proto-rules.html)

or look at a specific language feature

* [assignment]()
* [`class`](https://trree.gitbooks.io/cppcoreguidelines/content/c-classes-and-class-hierarchies.html)
* [constructor](https://trree.gitbooks.io/cppcoreguidelines/content/c-classes-and-class-hierarchies.html)
* [derived`class`](https://trree.gitbooks.io/cppcoreguidelines/content/c-classes-and-class-hierarchies.html)
* [destructor](https://trree.gitbooks.io/cppcoreguidelines/content/c-classes-and-class-hierarchies.html)
* [exception]()
* [`for`]()
* [`inline`]()
* [initialization]()
* [lambda expression]()
* [operator]()
* [`public`,`private`, and`protected`]()
* [`static_assert`]()
* [`struct`]()
* [`template`]()
* [`unsigned`]()
* [`virtual`]()

Definitions of terms used to express and discuss the rules, that are not language-technical, but refer to design and programming techniques

* error
* exception
* failure
* invariant
* leak
* precondition
* postcondition
* resource
* exception guarantee

# Abstract {#abstract}

This document is a set of guidelines for using C++ well. The aim of this document is to help people to use modern C++ effectively. By “modern C++” we mean C++11 and C++14 \(and soon C++17\). In other words, what would you like your code to look like in 5 years’ time, given that you can start now? In 10 years’ time?

The guidelines are focused on relatively higher-level issues, such as interfaces, resource management, memory management, and concurrency. Such rules affect application architecture and library design. Following the rules will lead to code that is statically type safe, has no resource leaks, and catches many more programming logic errors than is common in code today. And it will run fast – you can afford to do things right.

We are less concerned with low-level issues, such as naming conventions and indentation style. However, no topic that can help a programmer is out of bounds.

Our initial set of rules emphasizes safety \(of various forms\) and simplicity. They may very well be too strict. We expect to have to introduce more exceptions to better accommodate real-world needs. We also need more rules.

You will find some of the rules contrary to your expectations or even contrary to your experience. If we haven’t suggested you change your coding style in any way, we have failed! Please try to verify or disprove rules! In particular, we’d really like to have some of our rules backed up with measurements or better examples.

You will find some of the rules obvious or even trivial. Please remember that one purpose of a guideline is to help someone who is less experienced or coming from a different background or language to get up to speed.

Many of the rules are designed to be supported by an analysis tool. Violations of rules will be flagged with references \(or links\) to the relevant rule. We do not expect you to memorize all the rules before trying to write code. One way of thinking about these guidelines is as a specification for tools that happens to be readable by humans.

The rules are meant for gradual introduction into a code base. We plan to build tools for that and hope others will too.

Comments and suggestions for improvements are most welcome. We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.

