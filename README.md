# Table of Contents
* [Python Characteristics](#python-characteristics)

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




# References
1. 7 weeks for 7 programming languages

1. 7 Concurrency Models in 7 Weeks: When Threads Unravel

1. 7 weeks for Database

1. Pragmatic programming languages

1. Pycon Russia Presentation: Hettinger

1. David Beazley GIL

1. Fluent Python

1. Python Cookbook

