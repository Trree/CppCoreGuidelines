# FAQ: Answers to frequently asked questions {#faq-answers-to-frequently-asked-questions}

This section covers answers to frequently asked questions about these guidelines.

### FAQ.1: What do these guidelines aim to achieve? {#faq1-what-do-these-guidelines-aim-to-achieve}

See the[top of this page](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-abstract). This is an open source project to maintain modern authoritative guidelines for writing C++ code using the current C++ Standard \(as of this writing, C++14\). The guidelines are designed to be modern, machine-enforceable wherever possible, and open to contributions and forking so that organizations can easily incorporate them into their own corporate coding guidelines.

### FAQ.2: When and where was this work first announced? {#faq2-when-and-where-was-this-work-first-announced}

It was announced by[Bjarne Stroustrup in his CppCon 2015 opening keynote, “Writing Good C++14”](https://isocpp.org/blog/2015/09/stroustrup-cppcon15-keynote). See also the[accompanying isocpp.org blog post](https://isocpp.org/blog/2015/09/bjarne-stroustrup-announces-cpp-core-guidelines), and for the rationale of the type and memory safety guidelines see[Herb Sutter’s follow-up CppCon 2015 talk, “Writing Good C++14 … By Default”](https://isocpp.org/blog/2015/09/sutter-cppcon15-day2plenary).

### FAQ.3: Who are the authors and maintainers of these guidelines? {#faq3-who-are-the-authors-and-maintainers-of-these-guidelines}

The initial primary authors and maintainers are Bjarne Stroustrup and Herb Sutter, and the guidelines so far were developed with contributions from experts at CERN, Microsoft, Morgan Stanley, and several other organizations. At the time of their release, the guidelines are in a “0.6” state, and contributions are welcome. As Stroustrup said in his announcement: “We need help!”

### FAQ.4: How can I contribute? {#faq4-how-can-i-contribute}

See[CONTRIBUTING.md](https://github.com/isocpp/CppCoreGuidelines/blob/master/CONTRIBUTING.md). We appreciate volunteer help!

### FAQ.5: How can I become an editor/maintainer? {#faq5-how-can-i-become-an-editormaintainer}

By contributing a lot first and having the consistent quality of your contributions recognized. See[CONTRIBUTING.md](https://github.com/isocpp/CppCoreGuidelines/blob/master/CONTRIBUTING.md). We appreciate volunteer help!

### FAQ.6: Have these guidelines been approved by the ISO C++ standards committee? Do they represent the consensus of the committee? {#faq6-have-these-guidelines-been-approved-by-the-iso-c-standards-committee-do-they-represent-the-consensus-of-the-committee}

No. These guidelines are outside the standard. They are intended to serve the standard, and be maintained as current guidelines about how to use the current Standard C++ effectively. We aim to keep them in sync with the standard as that is evolved by the committee.

### FAQ.7: If these guidelines are not approved by the committee, why are they under`github.com/isocpp`? {#faq7-if-these-guidelines-are-not-approved-by-the-committee-why-are-they-under-githubcomisocpp}

Because`isocpp`is the Standard C++ Foundation; the committee’s repositories are under[github.com/_cplusplus_](https://github.com/cplusplus). Some neutral organization has to own the copyright and license to make it clear this is not being dominated by any one person or vendor. The natural entity is the Foundation, which exists to promote the use and up-to-date understanding of modern Standard C++ and the work of the committee. This follows the same pattern that isocpp.org did for the[C++ FAQ](https://isocpp.org/faq), which was initially the work of Bjarne Stroustrup, Marshall Cline, and Herb Sutter and contributed to the open project in the same way.

### FAQ.8: Will there be a C++98 version of these Guidelines? a C++11 version? {#faq8-will-there-be-a-c98-version-of-these-guidelines-a-c11-version}

No. These guidelines are about how to best use Standard C++14 \(and, if you have an implementation available, the Concepts Lite Technical Specification\) and write code assuming you have a modern conforming compiler.

### FAQ.9: Do these guidelines propose new language features? {#faq9-do-these-guidelines-propose-new-language-features}

No. These guidelines are about how to best use Standard C++14 + the Concepts Lite Technical Specification, and they limit themselves to recommending only those features.

### FAQ.10: What version of Markdown do these guidelines use? {#faq10-what-version-of-markdown-do-these-guidelines-use}

These coding standards are written using[CommonMark](http://commonmark.org/), and`<a>`HTML anchors.

We are considering the following extensions from[GitHub Flavored Markdown \(GFM\)](https://help.github.com/articles/github-flavored-markdown/):

* fenced code blocks \(consistently using indented vs. fenced is under discussion\)
* tables \(none yet but we’ll likely need them, and this is a GFM extension\)

Avoid other HTML tags and other extensions.

Note: We are not yet consistent with this style.

### FAQ.50: What is the GSL \(guideline support library\)? {#faq50-what-is-the-gsl-guideline-support-library}

The GSL is the small set of types and aliases specified in these guidelines. As of this writing, their specification herein is too sparse; we plan to add a WG21-style interface specification to ensure that different implementations agree, and to propose as a contribution for possible standardization, subject as usual to whatever the committee decides to accept/improve/alter/reject.

### FAQ.51: Is[github.com/Microsoft/GSL](https://github.com/Microsoft/GSL)the GSL? {#faq51-is-githubcommicrosoftgsl-the-gsl}

No. That is just a first implementation contributed by Microsoft. Other implementations by other vendors are encouraged, as are forks of and contributions to that implementation. As of this writing one week into the public project, at least one GPLv3 open source implementation already exists. We plan to produce a WG21-style interface specification to ensure that different implementations agree.

### FAQ.52: Why not supply an actual GSL implementation in/with these guidelines? {#faq52-why-not-supply-an-actual-gsl-implementation-inwith-these-guidelines}

We are reluctant to bless one particular implementation because we do not want to make people think there is only one, and inadvertently stifle parallel implementations. And if these guidelines included an actual implementation, then whoever contributed it could be mistakenly seen as too influential. We prefer to follow the long-standing approach of the committee, namely to specify interfaces, not implementations. But at the same time we want at least one implementation available; we hope for many.

### FAQ.53: Why weren’t the GSL types proposed through Boost? {#faq53-why-werent-the-gsl-types-proposed-through-boost}

Because we want to use them immediately, and because they are temporary in that we want to retire them as soon as types that fill the same needs exist in the standard library.

### FAQ.54: Has the GSL \(guideline support library\) been approved by the ISO C++ standards committee? {#faq54-has-the-gsl-guideline-support-library-been-approved-by-the-iso-c-standards-committee}

No. The GSL exists only to supply a few types and aliases that are not currently in the standard library. If the committee decides on standardized versions \(of these or other types that fill the same need\) then they can be removed from the GSL.

### FAQ.55: If you’re using the standard types where available, why is the GSL`string_span`different from the`string_view`in the Library Fundamentals 1 Technical Specification and C++17 Working Paper? Why not just use the committee-approved`string_view`? {#faq55-if-youre-using-the-standard-types-where-available-why-is-the-gsl-string_span-different-from-the-string_view-in-the-library-fundamentals-1-technical-specification-and-c17-working-paper-why-not-just-use-the-committee-approved-string_view}

The consensus on the taxonomy of views for the C++ standard library was that “view” means “read-only”, and “span” means “read/write”. The read-only`string_view`was the first such component to complete the standardization process, while`span`and`string_span`are currently being considered for standardization.

### FAQ.56: Is`owner`the same as the proposed`observer_ptr`? {#faq56-is-owner-the-same-as-the-proposed-observer_ptr}

No.`owner`owns, is an alias, and can be applied to any indirection type. The main intent of`observer_ptr`is to signify a_non_-owning pointer.

### FAQ.57: Is`stack_array`the same as the standard`array`? {#faq57-is-stack_array-the-same-as-the-standard-array}

No.`stack_array`is guaranteed to be allocated on the stack. Although a`std::array`contains its storage directly inside itself, the`array`object can be put anywhere, including the heap.

### FAQ.58: Is`dyn_array`the same as`vector`or the proposed`dynarray`? {#faq58-is-dyn_array-the-same-as-vector-or-the-proposed-dynarray}

No.`dyn_array`is not resizable, and is a safe way to refer to a heap-allocated fixed-size array. Unlike`vector`, it is intended to replace array-`new[]`. Unlike the`dynarray`that has been proposed in the committee, this does not anticipate compiler/language magic to somehow allocate it on the stack when it is a member of an object that is allocated on the stack; it simply refers to a “dynamic” or heap-based array.

### FAQ.59: Is`Expects`the same as`assert`? {#faq59-is-expects-the-same-as-assert}

No. It is a placeholder for language support for contract preconditions.

### FAQ.60: Is`Ensures`the same as`assert`? {#faq60-is-ensures-the-same-as-assert}

No. It is a placeholder for language support for contract postconditions.

  


