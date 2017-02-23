# C++ Core Guidelines 

April 3, 2017

Editors:

* [Bjarne Stroustrup](http://www.stroustrup.com/)
* [Herb Sutter](http://herbsutter.com/)

这个文档是非常早的一个草稿，它是不正确的，不完整的。现在它已经成为一个开源的项目，发布到 0.7 版本，这个项目使用的是 MIT-sytle license ，所以你可以复制，使用，修改和再创造。为此项目做出贡献需要同意授权许可。具体的查看附件[LICENSE](http://isocpp.github.io/CppCoreGuidelines/LICENSE).我们做这个项目提供给 “友好用户” 使用，复制，修改，再创造，并且希望有建设性的意见。

对于有益对评论和建议我们都非常欢迎。我们计划修改和拓展这篇文档，以提升我们对这门语音的理解和对可用库对改进。在你评论对时候，请注意[简介](/in-introduction.md)，这个大纲是我们的目标。大纲的贡献者在[这里](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-ack)

Problems:

* 这些规则还没有进行完整性，一致性，和可执行型测试
* 三个问号的标记 \(???\) 表示信息有待补充
* 更新引用的部分，许多c++11标准之前的代码太老了
* 对于计划性的工作请看 [To-do: Unclassified proto-rules](/to-do-unclassified-proto-rules.md)

你可以先看这个文档的[结构和解释](#S-abstract)，或者直接查看下面章节

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

支持部分:

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

或者查看特定的语言特性

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

用于表达和讨论的规则，那不是语言技术，而是指导设计和编程。

* error
* exception
* failure
* invariant
* leak
* precondition
* postcondition
* resource
* exception guarantee

# <span id="S-abstract"></span>Abstract 

这篇文档的一系列准则是为了更好的使用C++。目标是帮助人们更加高效的使用现代C++。“现代C++”指的是 C++11 ,C++14 和 C++17。换句话说，你会喜欢你现在写的代码还是5年前的样子，或者10年前的吗？

这些准则相对来说更集中在高级问题，比如接口，资源管理，内存管理和并发。这些规则都影响着应用构建和库的设计。遵守这些规则将使你的代码是静态类型安全，没有资源泄露，捕获更多的逻辑错误。

我们将更少的关心低层次的问题，像命名约定和代码风格。
然而，没有主题可以帮助程序员明知故犯。

我们最初设置规则是强调安全和简易。他们很可能是过于严格。我们希望介绍更多的例外去适应现实需求。此外，我们也需要更多的规则。

我们将搜寻一些与你直觉或者与你的经历相悖的规则。如果我们不能建议你修改你的代码风格，那么将是我们的失败。请尝试验证或反驳规则！特别说明，我们特别需要一些更好的备份数据和例子。

你会发现一些明了的，甚至琐碎的规则。请记住，一个指引的目的之一是帮助那些缺乏经验或从不同的背景或语言的人来加快学习速度。

许多规则旨在通过分析工具来支持。违反的规则将与相关规则一起被标记。 我们不指望你尝试写代码之前，记住所有的规则。思考这些指导方针的一种方式是作为工具的规范正好是人类可读的。

这些规则都是为了逐步引入一个代码库。我们计划建立这种工具，并希望其他人一起加入.

对于有益对评论和建议我们都非常欢迎。我们计划修改和拓展这篇文档，以提升我们对这门语音对理解和对可用库对改进。

