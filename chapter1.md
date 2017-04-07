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

* [In: Introduction](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-introduction)
* [P: Philosophy](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-philosophy)
* [I: Interfaces](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-interfaces)
* [F: Functions](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-functions)
* [C: Classes and class hierarchies](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-class)
* [Enum: Enumerations](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-enum)
* [R: Resource management](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource)
* [ES: Expressions and statements](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-expr)
* [Per: Performance](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-performance)
* [CP: Concurrency](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-concurrency)
* [E: Error handling](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-errors)
* [Con: Constants and immutability](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-const)
* [T: Templates and generic programming](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-templates)
* [CPL: C-style programming](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-cpl)
* [SF: Source files](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-source)
* [SL: The Standard library](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib)

Supporting sections:

* [A: Architectural Ideas](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-A)
* [N: Non-Rules and myths](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-not)
* [RF: References](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-references)
* [Pro: Profiles](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-profile)
* [GSL: Guideline support library](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-gsl)
* [NL: Naming and layout](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-naming)
* [FAQ: Answers to frequently asked questions](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-faq)
* [Appendix A: Libraries](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-libraries)
* [Appendix B: Modernizing code](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-modernizing)
* [Appendix C: Discussion](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-discussion)
* [Glossary](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-glossary)
* [To-do: Unclassified proto-rules](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-unclassified)

or look at a specific language feature

* [assignment](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [`class`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-class)
* [constructor](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-ctor)
* [derived`class`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-hier)
* [destructor](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-dtor)
* [exception](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-errors)
* [`for`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [`inline`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-class)
* [initialization](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [lambda expression](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-lambdas)
* [operator](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [`public`,`private`, and`protected`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [`static_assert`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [`struct`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-class)
* [`template`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [`unsigned`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-???)
* [`virtual`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-hier)

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

