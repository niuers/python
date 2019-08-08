# Table of Contents
* [Python Characteristics](#python-characteristics)
* [Characters and Bytes](#characters-and-bytes)

# Python Characteristics
### Interpreted rather than compiled
### Dynamic type system
### Pass by value with object references
### Modular capability
### Comprehensive libraries
### Extensibility with respect to other languages
### Object orientation
### Most of the major programming paradigms-procedural, object-oriented, and to a lesser extent, functional.

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

