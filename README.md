# Table of Contents
* [The Zen of Python](#the-zen-of-python)
* [Python Overview](#python-overview)
* [Python Data Model](#python-data-model)
* [Python Sequences](./python_sequences.md)
* [What is Pythonic?](#what-is-pythonic)
* [Python names and values](#python-names-and-values)
* [Libraries](#libraries)
* [Techniques](#techniques)
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
* Python adheres to the idea that, if a language behaves a certain way in some contexts, it should ideally work similarly in all contexts. 
* Python also follows the principle that a language should not have “convenient” shortcuts, special cases, ad hoc exceptions, overly subtle distinctions, or mysterious and tricky under-the-covers optimizations. A good language, like any other well-designed artifact, must balance such general principles with taste, common sense, and a high degree of practicality.

#### Very High Level Language (VHLL)
* Python is a very high-level language (VHLL). This means that Python uses a higher level of abstraction, conceptually further from the underlying machine, than do classic compiled languages such as C, C++, and Fortran, which are traditionally called “high-level languages.” 
* Python is simpler, faster to process (both for human brains and for programmatic tools), and more regular than classic high-level languages. 
* This enables high programmer productivity and makes Python an attractive development tool. Good compilers for classic compiled languages can generate binary machine code that runs faster than Python code. However, in most cases, the performance of Python-coded applications is sufficient. When it isn’t, apply the optimization techniques covered in “Optimization” to improve your program’s performance while keeping the benefit of high productivity.

* Somewhat newer languages such as Java and C# are slightly higher-level than classic ones such as C and Fortran, and share some characteristics of classic languages (such as the need to use declarations) as well as some of VHLLs like Python (such as the use of portable bytecode as the compilation target in typical implementations, and garbage collection to relieve programmers from the need to manage memory).
* In terms of language level, Python is comparable to other powerful VHLLs like JavaScript, Ruby, and Perl. The advantages of simplicity and regularity, however, remain on Python’s side.

#### Cross-Platform
#### Interpreted rather than compiled
#### Dynamic type system
#### Pass by value with object references
#### Modular capability
#### Comprehensive libraries
#### Extensibility with respect to other languages
#### Support most of the major programming paradigms
* Procedural
* Object-oriented
* To a lesser extent, functional.

## Python Implementations

* The primary difference between the implementations is the environment in which they run and the libraries and frameworks they can use.
* CPython applications are often faster than Jython or IronPython, particularly if you use extension modules such as NumPy (covered in “Array Processing”); however, PyPy can often be even faster, thanks to just-in-time compilation to machine code. It may be worthwhile to benchmark your CPython code against PyPy.

### CPython
* CPython is a compiler, interpreter, and set of built-in and optional extension modules, all coded in standard C.

### Jython
* Jython is a Python implementation for any Java Virtual Machine (JVM) compliant with Java 7 or better, written in Java.

### IronPython
* IronPython is a Python implementation for the Microsoft-designed Common Language Runtime (CLR), most commonly known as .NET, written in C#.

### PyPy
* PyPy is a fast and flexible implementation of Python, coded in a subset of Python itself, able to target several lower-level languages and virtual machines using advanced techniques such as type inferencing. PyPy’s greatest strength is its ability to generate native machine code “just in time” as it runs your Python program. 
* PyPy has substantial advantages in speed and memory management, and is seeing production-level use.

### Pyston
* A new high-performance implementation in early development

# Python Interpreter
* The Python interpreter program is run as **python** (it’s named python.exe on Windows). 
* **python** includes both the interpreter itself and the Python compiler, which is implicitly invoked, as and if needed, on imported modules.


# Python Data Model

### Python Data Model Overview
> The Python data model (or object model) is a description of Python as a framework. It formalizes the interfaces of the building blocks of the language itself, such as sequences, iterators, functions, classes, context managers, and so on. It's about “The properties of objects in general in a specific computer programming language.”

### Objects in Python

* Objects are Python's abstraction for Data. All data in a Python program is represented by objects or by relations between objects. Every object has an identity, a type and a value. 

#### Object Identity
* An object’s identity never changes once it has been created; you may think of it as the object’s address in memory (e.g. CPython). The `is` operator compares the identity of two objects; the `id()` function returns an integer representing its identity.

#### Object Type
* An object’s type determines the operations that the object supports (e.g., “does it have a length?”) and also defines the possible values for objects of that type. 
* The `type()` function returns an object’s type (which is an object itself). 
* Like its identity, an object’s type is also unchangeable (It is possible in some cases to change an object’s type, under certain controlled conditions.).
  
#### Object Value
* The value of some objects can change. (mutable vs. immutable objects)

#### Garbage Collection
* Objects are never explicitly destroyed; however, when they become unreachable they **may** be garbage-collected (not guaranteed and it's implementation dependent)
  * CPython currently uses a **reference-counting scheme** with (optional) **delayed detection of cyclically linked garbage**, which collects most objects as soon as they become unreachable, but is not guaranteed to collect garbage containing circular references. See the `gc` module. Do not depend on immediate finalization of objects when they become unreachable (so you should always close files explicitly).

#### Containers
* Containers contain references to other objects. The references are part of a container’s value. 
  * In most cases, when we talk about the value of a container, we imply the values, not the identities of the contained objects;   
  * However, when we talk about the mutability of a container, only the identities of the immediately contained objects are implied. So, if an immutable container (like a tuple) contains a reference to a mutable object, its value changes if that mutable object is changed.

#### Object Types Impact Identity
* Types affect almost all aspects of object behavior. Even the importance of object identity is affected in some sense.
  * For immutable types, operations that compute new values may actually return a reference to any existing object with the same type and value, while for mutable objects this is not allowed.
  * Example: after `a = 1; b = 1`, `a` and `b` may or may not refer to the same object with the value one, depending on the implementation, but after `c = []; d = []`, `c` and `d` are *guaranteed* to refer to two different, unique, newly created empty lists. (Note that `c = d = []` assigns the same object to both `c` and `d`.)

#### Data Types
* Python does not support single-precision floating point numbers.
* Immutable types
  * Strings
  * Tuples
  * Bytes
  * FrozenSet
* Mutable types
  * Lists
  * Set
  * Dict
    * The only types of values not acceptable as keys are values containing lists or dictionaries or other mutable types that are compared by value rather than by object identity, the reason being that the efficient implementation of dictionaries requires a key’s hash value to remain constant. Numeric types used for keys obey the normal rules for numeric comparison: if two numbers compare equal (e.g., 1 and 1.0) then they can be used interchangeably to index the same dictionary entry.
  * ByteArray



### Dunder Methods
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

### References
* [Alex Martelli](#alex-martellis-python-in-a-nutshell-2e)
* [Python Data Model](#https://docs.python.org/3/reference/datamodel.html)



# What is Pythonic?

1. Generic operations on sequence
   * Strings, lists, byte sequences, arrays, XML elements, and database results share a rich set of common operations including iteration, slicing, sorting, and concatenation.
   
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
* Immutable values: numbers, strings, bytes, and tuples. 
Immutable means that the value can never change, instead when you think you are changing the value, you are really making new values from old ones.
* Mutable values: Almost everything else is mutable, including lists, dicts, bytearray, array.array, collections.deque, memoryview and user-defined objects. 
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

# Libraries
## NumPy
* NumPy implements multi-dimensional, homogeneous arrays and matrix types that hold not only numbers but also user-defined records, and provides efficient elementwise operations.

## SciPy
* SciPy is a library, written on top of NumPy, offering many scientific computing algorithms from linear algebra, numerical calculus, and statistics. 
* SciPy is fast and reliable because it leverages the widely used C and Fortran code base from the Netlib Repository. 


# Techniques
* Check the byte code Python generated
```
>>> import dis
>>> dis.dis('a*2')
```
* An important Python API convention: functions or methods that change an object in place should return `None` to make it clear to the caller that the object itself was changed, and no new object was created.
  * list.sort, random.shuffle

* High-resolution performance measurement timer
```
from time import perf_counter as pc
t0 = pc(); floats /= 3; pc() - t0
```

* Convert a number to binary
```
bin(-1)
bin(7)
```

#### Others


# References
#### 7 weeks for 7 programming languages
1. 7 Concurrency Models in 7 Weeks: When Threads Unravel
1. 7 weeks for Database
1. Pragmatic programming languages
1. Pycon Russia Presentation: Hettinger
1. David Beazley GIL
1. Fluent Python
1. Python Cookbook
1. The Art of the Metaobject Protocol (AMOP)
#### 
Alex Martelli’s Python in a Nutshell 2E
#### 
David Beazley’s Python Essential Reference 4E
#### 
Expert Python Programming by Tarek Ziadé (Packt) is one of the best intermediate-level Python books in the market, and its final chapter, “Useful Design Patterns,” presents seven of the classic patterns from a Pythonic perspective.
#### 
Alex Martelli has given several talks about Python Design Patterns. There is a video of his EuroPython 2011 presentation and a set of slides on his personal website. I’ve found different slide decks and videos over the years, of varying lengths, so it is worthwhile to do a thorough search for his name with the words “Python Design Patterns.”
#### 
Around 2008, Bruce Eckel—author of the excellent Thinking in Java (Prentice Hall)—started a book titled Python 3 Patterns, Recipes and Idioms. It was to be written by a community of contributors led by Eckel, but six years later it’s still incomplete and apparently stalled (as I write this, the last change to the repository is two years old).
#### In Design Patterns in Dynamic Languages (slides), Peter Norvig shows how first-class functions (and other dynamic features) make several of the original design patterns either simpler or unnecessary.

####
* That is the source of the often quoted design principles “Program to an interface, not an implementation” and “Favor object composition over class inheritance.”
#### 
As far as I know, Learning Python Design Patterns, by Gennadiy Zlobin (Packt), is the only book entirely devoted to patterns in Python—as of June 2014. But Zlobin’s work is quite short (100 pages) and covers eight of the original 23 design patterns.

