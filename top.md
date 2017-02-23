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

* [in-introduction](/in-introduction.md)
* [p-philosophy](/p-philosophy.md)
* [I: Interfaces](/i-interfaces.md)
* [F: Functions](/f-functions.md)
* [C: Classes and Class Hierarchies](/c-classes-and-class-hierarchies.md)
* [Enum: Enumerations](/enum-enumerations.md)
* [R: Resource management](/r-resource-management.md)
* [ES: Expressions and Statements](/es-expressions-and-statements.md)
* [i-interfaces.md](/i-interfaces.md)
* [Per: Performance](/per-performance.md)
* [CP: Concurrency](/cp-concurrency-and-parallelism.md)
* [E: Error handling](/e-error-handling.md)
* [Con: Constants and immutability](/con-constants-and-immutability.md)
* [T: Templates and generic programming](/t-templates-and-generic-programming.md)
* [CPL: C-style programming](/cpl-c-style-programming.md)
* [SF: Source files](/sf-source-files.md)
* [SL: The Standard library](/sl-the-standard-library.md)

Supporting sections:

* [A: Architectural Ideas](/a-architectural-ideas.md)
* [N: Non-Rules and myths](/nr-non-rules-and-myths.md)
* [RF: References](/rf-references.md)
* [Pro: Profiles](/pro-profiles.md)
* [GSL: Guideline support library](/gsl-guideline-support-library.md)
* [NL: Naming and layout](/nl-naming-and-layout-rules.md)
* [FAQ: Answers to frequently asked questions](/faq-answers-to-frequently-asked-questions.md)
* [Appendix A: Libraries](/appendix-a-libraries.md)
* [Appendix B: Modernizing code](/appendix-b-modernizing-code.md)
* [Appendix C: Discussion](/appendix-c-discussion.md)
* [Glossary](/glossary.md)
* [To-do: Unclassified proto-rules](/to-do-unclassified-proto-rules.md)

or look at a specific language feature

* [assignment]()
* [`class`](/c-classes-and-class-hierarchies.md)
* [constructor](/c-classes-and-class-hierarchies.md)
* [derived`class`](/c-classes-and-class-hierarchies.md)
* [destructor](/c-classes-and-class-hierarchies.md)
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

