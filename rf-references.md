# <span id="S-references"></span>RF: References

Many coding standards, rules, and guidelines have been written for C++, and especially for specialized uses of C++.
Many

* focus on lower-level issues, such as the spelling of identifiers
* are written by C++ novices
* see "stopping programmers from doing unusual things" as their primary aim
* aim at portability across many compilers (some 10 years old)
* are written to preserve decades old code bases
* aim at a single application domain
* are downright counterproductive
* are ignored (must be ignored by programmers to get their work done well)

A bad coding standard is worse than no coding standard.
However an appropriate set of guidelines are much better than no standards: "Form is liberating."

Why can't we just have a language that allows all we want and disallows all we don't want ("a perfect language")?
Fundamentally, because affordable languages (and their tool chains) also serve people with needs that differ from yours and serve more needs than you have today.
Also, your needs change over time and a general-purpose language is needed to allow you to adapt.
A language that is ideal for today would be overly restrictive tomorrow.

Coding guidelines adapt the use of a language to specific needs.
Thus, there cannot be a single coding style for everybody.
We expect different organizations to provide additions, typically with more restrictions and firmer style rules.

Reference sections:

* [RF.rules: Coding rules](#SS-rules)
* [RF.books: Books with coding guidelines](#SS-books)
* [RF.C++: C++ Programming (C++11/C++14)](#SS-Cplusplus)
* [RF.web: Websites](#SS-web)
* [RS.video: Videos about "modern C++"](#SS-vid)
* [RF.man: Manuals](#SS-man)

## <span id="SS-rules"></span>RF.rules: Coding rules

* [Boost Library Requirements and Guidelines](http://www.boost.org/development/requirements.html).
  ???.
* [Bloomberg: BDE C++ Coding](https://github.com/bloomberg/bde/wiki/CodingStandards.pdf).
  Has a strong emphasis on code organization and layout.
* Facebook: ???
* [GCC Coding Conventions](https://gcc.gnu.org/codingconventions.html).
  C++03 and (reasonably) a bit backwards looking.
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
  Geared toward C++03 and (also) older code bases. Google experts are now actively collaborating here on helping to improve these Guidelines, and hopefully to merge efforts so these can be a modern common set they could also recommend.
* [JSF++: JOINT STRIKE FIGHTER AIR VEHICLE C++ CODING STANDARDS](http://www.stroustrup.com/JSF-AV-rules.pdf).
  Document Number 2RDU00001 Rev C. December 2005.
  For flight control software.
  For hard real time.
  This means that it is necessarily very restrictive ("if the program fails somebody dies").
  For example, no free store allocation or deallocation may occur after the plane takes off (no memory overflow and no fragmentation allowed).
  No exception may be used (because there was no available tool for guaranteeing that an exception would be handled within a fixed short time).
  Libraries used have to have been approved for mission critical applications.
  Any similarities to this set of guidelines are unsurprising because Bjarne Stroustrup was an author of JSF++.
  Recommended, but note its very specific focus.
* [Mozilla Portability Guide](https://developer.mozilla.org/en-US/docs/Mozilla/C%2B%2B_Portability_Guide).
  As the name indicates, this aims for portability across many (old) compilers.
  As such, it is restrictive.
* [Geosoft.no: C++ Programming Style Guidelines](http://geosoft.no/development/cppstyle.html).
  ???.
* [Possibility.com: C++ Coding Standard](http://www.possibility.com/Cpp/CppCodingStandard.html).
  ???.
* [SEI CERT: Secure C++ Coding Standard](https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=637).
  A very nicely done set of rules (with examples and rationales) done for security-sensitive code.
  Many of their rules apply generally.
* [High Integrity C++ Coding Standard](http://www.codingstandard.com/).
* [llvm](http://llvm.org/docs/CodingStandards.html).
  Somewhat brief, pre-C++11, and (not unreasonably) adjusted to its domain.
* ???

## <span id="SS-books"></span>RF.books: Books with coding guidelines

* [Meyers96](#Meyers96) Scott Meyers: *More Effective C++*. Addison-Wesley 1996.
* [Meyers97](#Meyers97) Scott Meyers: *Effective C++, Second Edition*. Addison-Wesley 1997.
* [Meyers01](#Meyers01) Scott Meyers: *Effective STL*. Addison-Wesley 2001.
* [Meyers05](#Meyers05) Scott Meyers: *Effective C++, Third Edition*. Addison-Wesley 2005.
* [Meyers15](#Meyers15) Scott Meyers: *Effective Modern C++*. O'Reilly 2015.
* [SuttAlex05](#SuttAlex05) Sutter and Alexandrescu: *C++ Coding Standards*. Addison-Wesley 2005. More a set of meta-rules than a set of rules. Pre-C++11.
* [Stroustrup05](#Stroustrup05) Bjarne Stroustrup: [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf).
  LCSD05. October 2005.
* [Stroustrup14](#Stroustrup05) Stroustrup: [A Tour of C++](http://www.stroustrup.com/Tour.html).
  Addison Wesley 2014.
  Each chapter ends with an advice section consisting of a set of recommendations.
* [Stroustrup13](#Stroustrup13) Stroustrup: [The C++ Programming Language (4th Edition)](http://www.stroustrup.com/4th.html).
  Addison Wesley 2013.
  Each chapter ends with an advice section consisting of a set of recommendations.
* Stroustrup: [Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)
  for [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html).
  Mostly low-level naming and layout rules.
  Primarily a teaching tool.

## <span id="SS-Cplusplus"></span>RF.C++: C++ Programming (C++11/C++14)

* [TC++PL4](http://www.stroustrup.com/4th.html):
A thorough description of the C++ language and standard libraries for experienced programmers.
* [Tour++](http://www.stroustrup.com/Tour.html):
An overview of the C++ language and standard libraries for experienced programmers.
* [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html):
A textbook for beginners and relative novices.

## <span id="SS-web"></span>RF.web: Websites

* [isocpp.org](https://isocpp.org)
* [Bjarne Stroustrup's home pages](http://www.stroustrup.com)
* [WG21](http://www.open-std.org/jtc1/sc22/wg21/)
* [Boost](http://www.boost.org)<span id="Boost"></span>
* [Adobe open source](http://www.adobe.com/open-source.html)
* [Poco libraries](http://pocoproject.org/)
* Sutter's Mill?
* ???

## <span id="SS-vid"></span>RS.video: Videos about "modern C++"

* Bjarne Stroustrup: [C++11 Style](http://channel9.msdn.com/Events/GoingNative/GoingNative-2012/Keynote-Bjarne-Stroustrup-Cpp11-Style). 2012.
* Bjarne Stroustrup: [The Essence of C++: With Examples in C++84, C++98, C++11, and C++14](http://channel9.msdn.com/Events/GoingNative/2013/Opening-Keynote-Bjarne-Stroustrup). 2013
* All the talks from [CppCon '14](https://isocpp.org/blog/2014/11/cppcon-videos-c9)
* Bjarne Stroustrup: [The essence of C++](https://www.youtube.com/watch?v=86xWVb4XIyE) at the University of Edinburgh. 2014.
* Sutter: ???
* CppCon 15
* ??? C++ Next
* ??? Meting C++
* ??? more ???

## <span id="SS-man"></span>RF.man: Manuals

* ISO C++ Standard C++11.
* ISO C++ Standard C++14.
* [ISO C++ Standard C++17 CD](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4606.pdf). Committee Draft.
* [Palo Alto "Concepts" TR](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf).
* [ISO C++ Concepts TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
* [WG21 Ranges report](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf). Draft.

## <span id="SS-ack"></span>Acknowledgements

Thanks to the many people who contributed rules, suggestions, supporting information, references, etc.:

* Peter Juhl
* Neil MacIntosh
* Axel Naumann
* Andrew Pardoe
* Gabriel Dos Reis
* Zhuang, Jiangang (Jeff)
* Sergey Zubkov

and see the contributor list on the github.

