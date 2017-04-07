# Enum: Enumerations {#enum-enumerations}

Enumerations are used to define sets of integer values and for defining types for such sets of values. There are two kind of enumerations, “plain”`enum`s and`class enum`s.

Enumeration rule summary:

* [Enum.1: Prefer enumerations over macros](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-macro)
* [Enum.2: Use enumerations to represent sets of related named constants](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-set)
* [Enum.3: Prefer`enum class`es over “plain”`enum`s](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-class)
* [Enum.4: Define operations on enumerations for safe and simple use](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-oper)
* [Enum.5: Don’t use`ALL_CAPS`for enumerators](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-caps)
* [Enum.6: Avoid unnamed enumerations](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-unnamed)
* [Enum.7: Specify the underlying type of an enumeration only when necessary](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-underlying)
* [Enum.8: Specify enumerator values only when necessary](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Renum-value)

### Enum.1: Prefer enumerations over macros {#enum1-prefer-enumerations-over-macros}

##### Reason {#reason-156}

Macros do not obey scope and type rules. Also, macro names are removed during preprocessing and so usually don’t appear in tools like debuggers.

##### Example {#example-134}

First some bad old code:

```
// webcolors.h (third party header)
#define RED   0xFF0000
#define GREEN 0x00FF00
#define BLUE  0x0000FF

// productinfo.h
// The following define product subtypes based on color
#define RED    0
#define PURPLE 1
#define BLUE   2

int webby = BLUE;   // webby == 2; probably not what was desired

```

Instead use an`enum`:

```
enum class Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
enum class Product_info { red = 0, purple = 1, blue = 2 };

int webby = blue;   // error: be specific
Web_color webby = Web_color::blue;

```

We used an`enum class`to avoid name clashes.

##### Enforcement {#enforcement-149}

Flag macros that define integer values.

### Enum.2: Use enumerations to represent sets of related named constants {#enum2-use-enumerations-to-represent-sets-of-related-named-constants}

##### Reason {#reason-157}

An enumeration shows the enumerators to be related and can be a named type.

##### Example {#example-135}

```
enum class Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };

```

##### Note {#note-151}

Switching on an enumeration is common and the compiler can warn against unusual patterns of case labels. For example:

```
enum class Product_info { red = 0, purple = 1, blue = 2 };

void print(Product_info inf)
{
    switch (inf) {
    case Product_info::red: cout 
<
<
 "red"; break;
    case Product_info::purple: cout 
<
<
 "purple"; break;
    }
}

```

Such off-by-one switch\`statements are often the results of an added enumerator and insufficient testing.

##### Enforcement {#enforcement-150}

* Flag
  `switch`
  -statements where the
  `case`
  s cover most but not all enumerators of an enumeration.
* Flag
  `switch`
  -statements where the
  `case`
  s cover a few enumerators of an enumeration, but has no
  `default`
  .

### Enum.3: Prefer class enums over “plain” enums {#enum3-prefer-class-enums-over-plain-enums}

##### Reason {#reason-158}

To minimize surprises: traditional enums convert to int too readily.

##### Example {#example-136}

```
void Print_color(int color);

enum Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
enum Product_info { Red = 0, Purple = 1, Blue = 2 };

Web_color webby = Web_color::blue;

// Clearly at least one of these calls is buggy.
Print_color(webby);
Print_color(Product_info::Blue);

```

Instead use an`enum class`:

```
void Print_color(int color);

enum class Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
enum class Product_info { red = 0, purple = 1, blue = 2 };

Web_color webby = Web_color::blue;
Print_color(webby);  // Error: cannot convert Web_color to int.
Print_color(Product_info::Red);  // Error: cannot convert Product_info to int.

```

##### Enforcement {#enforcement-151}

\(Simple\) Warn on any non-class`enum`definition.

### Enum.4: Define operations on enumerations for safe and simple use {#enum4-define-operations-on-enumerations-for-safe-and-simple-use}

##### Reason {#reason-159}

Convenience of use and avoidance of errors.

##### Example {#example-137}

```
enum class Day { mon, tue, wed, thu, fri, sat, sun };

Day operator++(Day
&
 d)
{
    return d == Day::sun ? Day::mon : Day{++d};
}

Day today = Day::sat;
Day tomorrow = ++today;

```

##### Enforcement {#enforcement-152}

Flag repeated expressions cast back into an enumeration.

### Enum.5: Don’t use`ALL_CAPS`for enumerators {#enum5-dont-use-all_caps-for-enumerators}

##### Reason {#reason-160}

Avoid clashes with macros.

##### Example, bad {#example-bad-62}

```
 // webcolors.h (third party header)
#define RED   0xFF0000
#define GREEN 0x00FF00
#define BLUE  0x0000FF

// productinfo.h
// The following define product subtypes based on color

enum class Product_info { RED, PURPLE, BLUE };   // syntax error

```

##### Enforcement {#enforcement-153}

Flag ALL\_CAPS enumerators.

### Enum.6: Avoid unnamed enumerations {#enum6-avoid-unnamed-enumerations}

##### Reason {#reason-161}

If you can’t name an enumeration, the values are not related

##### Example, bad {#example-bad-63}

```
enum { red = 0xFF0000, scale = 4, is_signed = 1 };

```

Such code is not uncommon in code written before there were convenient alternative ways of specifying integer constants.

##### Alternative {#alternative-6}

Use`constexpr`values instead. For example:

```
constexpr int red = 0xFF0000;
constexpr short scale = 4;
constexpr bool is_signed = true;

```

##### Enforcement {#enforcement-154}

Flag unnamed enumerations.

### Enum.7: Specify the underlying type of an enumeration only when necessary {#enum7-specify-the-underlying-type-of-an-enumeration-only-when-necessary}

##### Reason {#reason-162}

The default is the easiest to read and write.`int`is the default integer type.`int`is compatible with C`enum`s.

##### Example {#example-138}

```
enum class Direction : char { n, s, e, w,
                              ne, nw, se, sw };  // underlying type saves space

enum class Web_color : int { red   = 0xFF0000,
                             green = 0x00FF00,
                             blue  = 0x0000FF };  // underlying type is redundant

```

##### Note {#note-152}

Specifying the underlying type is necessary in forward declarations of enumerations:

```
enum Flags : char;

void f(Flags);

// ....

enum flags : char { /* ... */ };

```

##### Enforcement {#enforcement-155}

????

### Enum.8: Specify enumerator values only when necessary {#enum8-specify-enumerator-values-only-when-necessary}

##### Reason {#reason-163}

It’s the simplest. It avoids duplicate enumerator values. The default gives a consecutive set of values that is good for`switch`-statement implementations.

##### Example {#example-139}

```
enum class Col1 { red, yellow, blue };
enum class Col2 { red = 1, yellow = 2, blue = 2 }; // typo
enum class Month { jan = 1, feb, mar, apr, may, jun,
                   jul, august, sep, oct, nov, dec }; // starting with 1 is conventional
enum class Base_flag { dec = 1, oct = dec 
<
<
 1, hex = dec 
<
<
 2 }; // set of bits

```

Specifying values is necessary to match conventional values \(e.g.,`Month`\) and where consecutive values are undesirable \(e.g., to get separate bits as in`Base_flag`\).

##### Enforcement {#enforcement-156}

* Flag duplicate enumerator values
* Flag explicitly specified all-consecutive enumerator values



