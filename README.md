# Table of Contents
* [The Zen of Python](#the-zen-of-python)
* [Python Overview](#python-overview)
* [Python Data Model](#python-data-model)
* [Characters and Bytes](#characters-and-bytes)
* [What is Pythonic?](#what-is-pythonic)
* [Python names and values](#python-names-and-values)
* [References](#references)

# [The Zen of Python](https://www.python.org/doc/humor/#the-zen-of-python)
* Beautiful is better than ugly.
* Explicit is better than implicit.
* Simple is better than complex.
* Complex is better than complicated.
* Flat is better than nested.
* Sparse is better than dense.
* Readability counts.
* Special cases aren't special enough to break the rules.
* Although practicality beats purity
* Errors should never pass silently.
* Unless explicitly silenced.
* In the face of ambiguity, refuse the temptation to guess.
* There should be one-- and preferably only one --obvious way to do it.
* Although that way may not be obvious at first unless you're Dutch.
* Now is better than never.
* Although never is often better than right now.
* If the implementation is hard to explain, it's a bad idea.
* If the implementation is easy to explain, it may be a good idea.
* Namespaces are one honking great idea -- let's do more of those!

—Tim Peters

# Python Overview
* One of the best qualities of Python is its consistency.

## Python Characteristics
#### Interpreted rather than compiled
#### Dynamic type system
#### Pass by value with object references
#### Modular capability
#### Comprehensive libraries
#### Extensibility with respect to other languages
#### Object orientation
#### Most of the major programming paradigms-procedural, object-oriented, and to a lesser extent, functional.


# Python Data Model

### Python Data Model Overview
> The data model is a description of Python as a framework. It formalizes the interfaces of the building blocks of the language itself, such as sequences, iterators, functions, classes, context managers, and so on. It's about “The properties of objects in general in a specific computer programming language.”

* The **Python interpreter** invokes special methods (Dunder methods or Magic methods) to perform basic object operations, often triggered by special syntax. e.g., `__getitem__` ("dunder-getitem").
  * For example, in order to evaluate `my_collection[key]`, the interpreter calls `my_collection.__getitem__(key)`.

* Advantages of Python Data Model
  * The users of your classes don’t have to memorize arbitrary method names for standard operations (.size()? .len()? )
  * It’s easier to benefit from the core language features (e.g., iteration and slicing) and the rich Python standard library (and avoid reinventing the wheel, like the `random.choice function`).

### How Special Methods Are Used
* The special methods are meant to be called by the Python interpreter, and not by you.
  * But for built-in types like `list`, `str`, `bytearray`, and so on, the interpreter takes a shortcut: the CPython implementation of `len()` actually returns the value of the `ob_size` field in the `PyVarObject C struct` that represents any variable-sized built-in object in memory. This is much faster than calling a method.
  
* More often than not, the special method call is implicit. 
  * For example, the statement `for i in x:` actually causes the invocation of `iter(x)`, which in turn may call `x.__iter__()` if that is available.
  
* Normally, your code should not have many direct calls to special methods. Unless you are doing a lot of metaprogramming, you should be implementing special methods more often than invoking them explicitly. 

### Metaobject Protocol

* The **metaobject** part refers to the objects that are the building blocks of the language itself. In this context, **protocol** is a synonym of interface. So a metaobject protocol is a fancy synonym for **object model**: an API for core language constructs.
* A rich metaobject protocol enables extending a language to support new programming paradigms.

### Comparison with Object Oriented Language (OOL)
* `collection.len()` in OOL vs. `len(collection)` in Python

Given a collection of items, C++ and Python use different syntax to obtain the size of the collection.

```
# C++
std::vector<int> myints;
std::cout << "size: " << myints.size() << '\n';

# Python
myints = [1,2,3]
print("size: ", len(myints))
```



# Characters and Bytes
> String: String is a sequence of characters.

* The definition of characters here are "unicode characters" for python 3.x. The items you get out of `str` are unicode characters.
  ```
  s = 'café'
  len(s) # returns 4, meaning 4 characters
  ```
* 1 character != 1 byte  

## Characters

#### The Identity of Characters and Specific Byte Representations

* The Unicode standard explicitly separates the identity of characters from specific byte representations:
  * The identity of a character—its **code point**—is a number from 0 to 1,114,111 (base 10), shown in the Unicode standard as 4 to 6 hexadecimal digits with a “U+” prefix. For example, the code point for the letter A is U+0041, the Euro sign is U+20AC, and the musical symbol G clef is assigned to code point U+1D11E. About 10% of the valid code points have characters assigned to them in Unicode 6.3, the standard used in Python 3.4.
  
#### Encoding and Decoding

* The actual bytes that represent a character depend on the encoding in use.
* An **encoding** is an algorithm that converts code points to byte sequences; converting from bytes to code points is **decoding**.
  * The code point for A (U+0041) is encoded as the single byte \x41 in the UTF-8 encoding, or as the bytes \x41\x00 in UTF-16LE encoding. 
  * As another example, the Euro sign (U+20AC) becomes three bytes in UTF-8—\xe2\x82\xac—but in UTF-16LE it is encoded as two bytes: \xac\x20.
  ```
  b = s.encode('utf-8') #encode to bytes
  len(b) # returns 5
  b.decode('utf-8') #decoding
  ```

## The Python 3.x Bytes type

* Two basic built-in types for binary sequences
  * The immutable `bytes` type
  * The mutable `bytearray` type

* Each item in `bytes` or `bytearray` is an integer from 0 to 255, and not a one-character string like in the Python 2 `str`. However, a slice of a binary sequence always produces a binary sequence of the same type—including slices of length 1.

* Although binary sequences are really sequences of integers, their literal notation reflects the fact that ASCII text is often embedded in them. Therefore, three different displays are used, depending on each byte value:
  * For bytes in the printable ASCII range—from space to ~ —the ASCII character itself is used.
  * For bytes corresponding to tab, newline, carriage return, and \, the escape sequences \t, \n, \r, and \\ are used.
  * For every other byte value, a hexadecimal escape sequence is used (e.g., \x00 is the null byte).

  ```
  cafe = bytes('café', encoding='utf_8')
  cafe
  b'caf\xc3\xa9'
  ```
  
* Both `bytes` and `bytearray` support every `str` method except those that do formatting (format, format_map) and a few others that depend on Unicode data.

* Building a binary sequence from a buffer-like object (An object that implements the buffer protocol, e.g., bytes, bytearray, memoryview, array.array) is a low-level operation that may involve type casting. 
* Creating a bytes or bytearray object from any buffer-like source will always copy the bytes. In contrast, memoryview objects let you share memory between binary data structures. To extract structured information from binary sequences, the struct module is invaluable.

### STRUCTS AND MEMORY VIEWS
* The struct module provides functions to parse packed bytes into a tuple of fields of different types and to perform the opposite conversion, from a tuple into packed bytes. struct is used with bytes, bytearray, and memoryview objects.

> The memoryview class does not let you create or store byte sequences, but provides shared memory access to slices of data from other binary sequences, packed arrays, and buffers such as Python Imaging Library (PIL) images, without copying the bytes.

### Basic Encoders/Decoders
> codecs are encoder/decoder for text to byte conversion and vice versa.

* Garbled characters (the result of text being decoded using an unintended character encoding) are known as gremlins or mojibake (文字化け—Japanese for “transformed text”). The result is a systematic replacement of symbols with completely unrelated ones, often from a different writing system.
*  “�” (code point U+FFFD), the official Unicode REPLACEMENT CHARACTER intended to represent unknown characters.

* UTF-8 is the default source encoding for Python 3, just as ASCII was the default for Python 2 (starting with 2.5). If you load a .py module containing non-UTF-8 data and no encoding declaration, you get a `SyntaxError`.

* HOW TO DISCOVER THE ENCODING OF A BYTE SEQUENCE? Use package `chardetect`
* We then considered the theory and practice of encoding detection in the absence of metadata: in theory, it can’t be done, but in practice the Chardet package pulls it off pretty well for a number of popular encodings

#### Byte Order Mark (BOM)
* Although binary sequences of encoded text usually don’t carry explicit hints of their encoding, the UTF formats may prepend a **byte order mark** to the textual content. 

```
>>> u16 = 'El Niño'.encode('utf_16')
>>> u16
b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```

The bytes are `b'\xff\xfe'`. That is a BOM, byte-order mark, denoting the “little-endian” byte ordering of the Intel CPU where the encoding was performed.

On a little-endian machine, for each code point the least significant byte comes first: the letter 'E', code point U+0045 (decimal 69), is encoded in byte offsets 2 and 3 as 69 and 0:

```
>>> list(u16)
[255, 254, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]
```

On a big-endian CPU, the encoding would be reversed; 'E' would be encoded as 0 and 69.

* To avoid confusion, the UTF-16 encoding prepends the text to be encoded with the special character **ZERO WIDTH NO-BREAK SPACE (U+FEFF)**, which is invisible. On a little-endian system, that is encoded as b'\xff\xfe' (decimal 255, 254). Because, by design, there is no U+FFFE character, the byte sequence b'\xff\xfe' must mean the ZERO WIDTH NO-BREAK SPACE on a little-endian encoding, so the codec knows which byte ordering to use.
* There is a variant of UTF-16, UTF-16LE, that is explicitly little-endian, and another one explicitly big-endian, UTF-16BE. If you use them, a BOM is not generated:

```
>>> u16le = 'El Niño'.encode('utf_16le')
>>> list(u16le)
[69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]
>>> u16be = 'El Niño'.encode('utf_16be')
>>> list(u16be)
[0, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111]
```

* If present, the BOM is supposed to be filtered by the UTF-16 codec, so that you only get the actual text contents of the file without the leading ZERO WIDTH NO-BREAK SPACE. The standard says that if a file is UTF-16 and has no BOM, it should be assumed to be UTF-16BE (big-endian). However, the Intel x86 architecture is little-endian, so there is plenty of little-endian UTF-16 with no BOM in the wild.

* This whole issue of endianness only affects encodings that use words of more than one byte, like UTF-16 and UTF-32. 
* One big advantage of UTF-8 is that it produces the same byte sequence regardless of machine endianness, so no BOM is needed.
  * Nevertheless, some Windows applications (notably Notepad) add the BOM to UTF-8 files anyway—and Excel depends on the BOM to detect a UTF-8 file, otherwise it assumes the content is encoded with a Windows codepage. 
  * The character U+FEFF encoded in UTF-8 is the three-byte sequence b'\xef\xbb\xbf'. So if a file starts with those three bytes, it is likely to be a UTF-8 file with a BOM. However, Python does not automatically assume a file is UTF-8 just because it starts with b'\xef\xbb\xbf'.


### The Unicode Sandwich
* The best practice for handling text is the “Unicode sandwich”. 
  * This means that bytes should be decoded to str as early as possible on input (e.g., when opening a file for reading). The “meat” of the sandwich is the business logic of your program, where text handling is done exclusively on str objects. You should never be encoding or decoding in the middle of other processing. On output, the str are encoded to bytes as late as possible.
  * Always are explicit about the encodings in your programs, you will avoid a lot of pain

* In the next section, we demonstrated opening text files, an easy task except for one pitfall: the encoding= keyword argument is not mandatory when you open a text file, but it should be. If you fail to specify the encoding, you end up with a program that manages to generate “plain text” that is incompatible across platforms, due to conflicting default encodings. 

### Unicode Text Normalization
* String comparisons are complicated by the fact that Unicode has combining characters: diacritics and other marks that attach to the preceding character, appearing as one when printed.

* Text comparisons are surprisingly complicated because Unicode provides multiple ways of representing some characters, so normalizing is a prerequisite to text matching.

* The code point U+0301 is the COMBINING ACUTE ACCENT. Using it after “e” renders “é”. In the Unicode standard, sequences like 'é' and 'e\u0301' are called “canonical equivalents,” and applications are supposed to treat them as the same. But Python sees two different sequences of code points, and considers them not equal.

* The solution is to use Unicode normalization, provided by the unicodedata.normalize function.

#### CASE FOLDING
* Case folding is essentially converting all text to lowercase, with some additional transformations. It is supported by the str.casefold() method.

#### EXTREME “NORMALIZATION”: TAKING OUT DIACRITICS

### Sorting Unicode Text
* Python sorts sequences of any type by comparing the items in each sequence one by one. For strings, this means comparing the code points. Unfortunately, this produces unacceptable results for anyone who uses non-ASCII characters.

* The standard way to sort non-ASCII text in Python is to use the locale.strxfrm function which, according to the locale module docs, “transforms a string to one that can be used in locale-aware comparisons.”
* To enable locale.strxfrm, you must first set a suitable locale for your application, and pray that the OS supports it. On GNU/Linux (Ubuntu 14.04) with the pt_BR locale. 
* So you need to call setlocale(LC_COLLATE, «your_locale») before using locale.strxfrm as the key when sorting.

* So the standard library solution to internationalized sorting works, but seems to be well supported only on GNU/Linux (perhaps also on Windows, if you are an expert). Even then, it depends on locale settings, creating deployment headaches.
Fortunately, there is a simpler solution: the PyUCA library, available on PyPI.

#### SORTING WITH THE UNICODE COLLATION ALGORITHM

### The Unicode Database
* (a source of metadata about every character)
* The Unicode standard provides an entire database—in the form of numerous structured text files—that includes not only the table mapping code points to character names, but also metadata about the individual characters and how they are related.

## Dual-Mode str and bytes APIs
* dual-mode APIs offering functions that accept `str` or `bytes` arguments with special handling depending on the type.
* Some examples are in the re and os modules.

## STR VERSUS BYTES ON OS FUNCTIONS
* The GNU/Linux kernel is not Unicode savvy, so in the real world you may find filenames made of byte sequences that are not valid in any sensible encoding scheme, and cannot be decoded to str. File servers with clients using a variety of OSes are particularly prone to this problem.

In order to work around this issue, all os module functions that accept filenames or pathnames take arguments as str or bytes.

## Others
### What Is “Plain Text”?

For anyone who deals with non-English text on a daily basis, “plain text” does not imply “ASCII.” The Unicode Glossary defines plain text like this:

Computer-encoded text that consists only of a sequence of code points from a given standard, with no other formatting or structural information.

That definition starts very well, but I don’t agree with the part after the comma. HTML is a great example of a plain-text format that carries formatting and structural information. But it’s still plain text because every byte in such a file is there to represent a text character, usually using UTF-8. There are no bytes with nontext meaning, as you can find in a .png or .xls document where most bytes represent packed binary values like RGB values and floating-point numbers. In plain text, numbers are represented as sequences of digit characters.

I am writing this book in a plain-text format called—ironically—AsciiDoc, which is part of the toolchain of O’Reilly’s excellent Atlas book publishing platform. AsciiDoc source files are plain text, but they are UTF-8, not ASCII. Otherwise, writing this chapter would have been really painful. Despite the name, AsciiDoc is just great.

The world of Unicode is constantly expanding and, at the edges, tool support is not always there. That’s why I had to use images for Figures 4-1, 4-3, and 4-4: not all characters I wanted to show were available in the fonts used to render the book. On the other hand, the Ubuntu 14.04 and OSX 10.9 terminals display them perfectly well—including the Japanese characters for the word “mojibake”: 文字化け.

Unicode Riddles

Imprecise qualifiers such as “often,” “most,” and “usually” seem to pop up whenever I write about Unicode normalization. I regret the lack of more definitive advice, but there are so many exceptions to the rules in Unicode that it is hard to be absolutely positive.

For example, the µ (micro sign) is considered a “compatibility character” but the Ω (ohm) and Å (Ångström) symbols are not. The difference has practical consequences: NFC normalization—recommended for text matching—replaces the Ω (ohm) by Ω (uppercase Grek omega) and the Å (Ångström) by Å (uppercase A with ring above). But as a “compatibility character” the µ (micro sign) is not replaced by the visually identical μ (lowercase Greek mu), except when the stronger NFKC or NFKD normalizations are applied, and these transformations are lossy.

I understand the µ (micro sign) is in Unicode because it appears in the latin1 encoding and replacing it with the Greek mu would break round-trip conversion. After all, that’s why the micro sign is a “compatibility character.” But if the ohm and Ångström symbols are not in Unicode for compatibility reasons, then why have them at all? There are already code points for the GREEK CAPITAL LETTER OMEGA and the LATIN CAPITAL LETTER A WITH RING ABOVE, which look the same and replace them on NFC normalization. Go figure.

My take after many hours studying Unicode: it is hugely complex and full of special cases, reflecting the wonderful variety of human languages and the politics of industry standards.

How Are str Represented in RAM?

The official Python docs avoid the issue of how the code points of a str are stored in memory. This is, after all, an implementation detail. In theory, it doesn’t matter: whatever the internal representation, every str must be encoded to bytes on output.

In memory, Python 3 stores each str as a sequence of code points using a fixed number of bytes per code point, to allow efficient direct access to any character or slice.

Before Python 3.3, CPython could be compiled to use either 16 or 32 bits per code point in RAM; the former was a “narrow build,” and the latter a “wide build.” To know which you have, check the value of sys.maxunicode: 65535 implies a “narrow build” that can’t handle code points above U+FFFF transparently. A “wide build” doesn’t have this limitation, but consumes a lot of memory: 4 bytes per character, even while the vast majority of code points for Chinese ideographs fit in 2 bytes. Neither option was great, so you had to choose depending on your needs.

Since Python 3.3, when creating a new str object, the interpreter checks the characters in it and chooses the most economic memory layout that is suitable for that particular str: if there are only characters in the latin1 range, that str will use just one byte per code point. Otherwise, 2 or 4 bytes per code point may be used, depending on the str. This is a simplification; for the full details, look up PEP 393 — Flexible String Representation.

The flexible string representation is similar to the way the int type works in Python 3: if the integer fits in a machine word, it is stored in one machine word. Otherwise, the interpreter switches to a variable-length representation like that of the Python 2 long type. It is nice to see the spread of good ideas.




# What is Pythonic?
1. Generic operations on sequence
1. Strong typing without variable declaration
1. An important Python API convention: functions or methods that change an object in place should return None to
make it clear to the caller that the object itself was changed, and no new object was created. e.g. list.sort()
1. Built-in tuple and mapping types
1. Structure by indentation

# [Python names and values](https://nedbatchelder.com/text/names.html#h_names_and_values)
## Python names are like C++ pointers
* Names refer to values or a name is a reference to a value
```
x = 23 # name “x” refers to the value 23, similar to C++ pointer
y = x # Many names can refer to one value, i.e. 23
x = 12 #Names are reassigned independently of other names.
```
* Values live until nothing refers to them. A value is 'garbage collected' (reclaimed) when there's no name refers to it. Think about 'reference counting' pattern in C++.

## Mutable and immutable values
* Immutable values: numbers, strings, and tuples. 
Immutable means that the value can never change, instead when you think you are changing the value, you are really making new values from old ones.
* Mutable values: Almost everything else is mutable, including lists, dicts, and user-defined objects. 
Mutable means that the value has methods that can change the value in-place. 

## Assignment never makes new values, it never copies data

```
x = 23 # Assignment makes the name on the left refer to the value on the right.
y = x

# same here, names 'nums' and 'tri' refer to the same list [1,2,3]
nums = [1,2,3]
tri = nums
```
* **Mutable Presto-Chango**: Changes in a mutable value are visible through all of its names

* Rebinding the name: assign a name, makes 'x' refer to a new value
```x = x + 1```

* Mutating the value: change a value
```
nums.append(4) #name 'tri' see the change as well
nums = nums + [4] #This is not mutation, it creates a new list and make nums refer to the new list. So it's rebinding.
```
* False: Python assigns mutable and immutable values differently.
All assignment works the same: it makes a name refer to a value.
```
# Lots of things are assignment, each of following lines is assignment to name 'X'.
X = ...
for X in ...
[... for X in ...]
(... for X in ...)
{... for X in ...}
class X(...):
def X(...):
def fn(X): ... ; fn(12)
with ... as X:
except ... as X:
import X
from ... import X
import ... as X
from ... import ... as X
```
* Python passes function arguments by assigning to them.

## References can be more than just names

* Python has a number of compound data structures each of which hold references to values: list elements, dictionary keys and values, object attributes, and so on. Each of those can be used on the left-hand side of an assignment. 
* Anything that can appear on the left-hand side of an assignment statement is a reference, and everywhere “name” appears can be substituted with “reference”. All of the rules here about names apply exactly the same to any of these references.
* If you have list elements referring to other mutable values, like sub-lists, it’s important to remember that the list elements are just references to values.

## Dynamic Typing

* Any name can refer to any value at any time.
* Names have no type, values have no scope.
When we say that a function has a local variable, we mean that the name is scoped to the function: you can’t use the name outside the function, and when the function returns, the name is destroyed. But if the name’s value has other references, it will live on beyond the function call. It is a local name, not a local value.
* Values can’t be deleted, only names can.

## Pass Argument
Python is neither pass-by-value nor pass-by-reference, it is [“pass-by-object-reference”](https://robertheaton.com/2014/02/09/pythons-pass-by-object-reference-as-explained-by-philip-k-dick/), i.e. object references are passed by value.

#### Others

* For every sequence in python (except `str`), `s[i]` returns one item, and `s[i:i+1]` returns a sequence of the same type with the `s[1]` item inside it. The only sequence type where `s[0] == s[:1]` is the `str` type.


# References
1. 7 weeks for 7 programming languages
1. 7 Concurrency Models in 7 Weeks: When Threads Unravel
1. 7 weeks for Database
1. Pragmatic programming languages
1. Pycon Russia Presentation: Hettinger
1. David Beazley GIL
1. Fluent Python
1. Python Cookbook
1. The Art of the Metaobject Protocol (AMOP)
