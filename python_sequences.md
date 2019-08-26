# Table of Contents
* [Classifications](#classifications)
* [List](#list)
* [Tuples](#tuples)
  * [Tuples as records](#tuples-as-records)
  * [TUPLES AS IMMUTABLE LISTS](#tuples-as-immutable-lists)
* [When a List Is Not the Answer](#when-a-list-is-not-the-answer)
* [Common Operations to Python Sequences](#common-operations-to-python-sequences)
  * [Slicing](#slicing)
  * [Using + and * with Sequences](#using--and--with-sequences)
  * [Augmented Assignment (`+=` or `*=`) with Sequences](#augmented-assignment--or--with-sequences)
    * [A += Assignment Puzzler](#a--assignment-puzzler)
  * [`list.sort` and the `sorted` Built-In Function](#listsort-and-the-sorted-built-in-function)

# Classifications
### Container Sequences vs. Flat Sequences
* Container Sequences
> list, tuple, and collections.deque can hold items of different types. Container sequences hold references to the objects they contain, which may be of any type.


* Flat Sequences
> str, bytes, bytearray, memoryview, and array.array hold items of one type. Flat sequences physically store the value of each item within its own memory space, and not as distinct objects. Thus, flat sequences are more compact, faster and easier to use, but they are limited to holding primitive values like characters, bytes, and numbers.

### Mutable Sequences vs. Immutable Sequences
* Mutable Sequences: list, bytearray, array.array, collections.deque, and memoryview
* Immutable sequences: tuple, str, and bytes

# List
### List Comprehension and Generator Expressions
* A listcomp is meant to do one thing only: to build a new list.
* List comprehensions, generator expressions, and their siblings set and dict comprehensions now have their own local scope, like functions. 
  * Variables assigned within the expression are local, but variables in the surrounding scope can still be referenced. Even better, the local variables do not mask the variables from the surrounding scope.
* Listcomps do everything the map and filter functions do, and with comparable or better performance.

#### Generator Expressions (GenExp)
* To fill up nonlist sequence types (e.g. tuples, arrays), a genexp is the way to go.
```
symbols = '$¢£¥€¤'
tuple(ord(symbol) for symbol in symbols)
```

* The generator expression feeds the for loop producing one item at a time. Using a generator expression would save the expense of building a list in memory just to feed the for loop.
```
colors = ['black', 'white']
sizes = ['S', 'M', 'L']
for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes):
    print(tshirt)
```
## List Techniques
### Sort two lists at the same time using the "decorate, sort, undecorate" idiom
```
list1, list2 = zip(*sorted(zip(list1, list2)))
```

# Tuples

* They can be used as immutable lists and also as records with no field names.

## Tuples as records
* Tuples hold records: each item in the tuple holds the data for one field and the position of the item gives its meaning.
* Unlike lists, tuples often hold items of different types. That is natural, considering that each item in a tuple is really a field, and each field type is independent of the others.

### Iterable/Tuple Unpacking
> Iterable unpacking works with any iterable object. The only requirement is that the iterable yields exactly one item per variable in the receiving tuple, unless you use a star (*) to capture excess items

* Parallel Assignment: The most visible form of tuple unpacking; that is, assigning items from an iterable to a tuple of variables
```
>>> lax_coordinates = (33.9425, -118.408056)
>>> latitude, longitude = lax_coordinates  # tuple unpacking
```
* Swapping the values of variables without using a temporary variable
* Another example is prefixing an argument with a star when calling a function
```
>>> divmod(20, 8)
(2, 4)
>>> t = (20, 8)
>>> divmod(*t)
(2, 4)

```
* Enable functions to return multiple values in a way that is convenient to the caller
* Using * to grab excess items
  * Another way of focusing on just some of the items when unpacking a tuple is to use the *
  * Defining function parameters with *args to grab arbitrary excess arguments is a classic Python feature.
  * This applies to parallel assignment. In the context of parallel assignment, the * prefix can be applied to exactly one variable, but it can appear in any position:
  ```
  >>> a, b, *rest = range(5)
  >>> a, b, rest
  (0, 1, [2, 3, 4])
  >>> a, *body, c, d = range(5)
  >>> a, body, c, d
  (0, [1, 2], 3, 4)
  ```

* Nested Tuple Unpacking
  * The tuple to receive an expression to unpack can have nested tuples, like (a, b, (c, d)), and Python will do the right thing if the expression matches the nesting structure.
  ```
  metro_areas = [
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
    ]
  for name, cc, pop, (latitude, longitude) in metro_areas:
     ...
  ```
### collections.namedtuple
* The collections.namedtuple function is a factory that produces subclasses of tuple enhanced with field names and a class name—which helps debugging.
* Instances of a class that you build with namedtuple take exactly the same amount of memory as tuples because the field names are stored in the class. They use less memory than a regular object because they don’t store attributes in a per-instance `__dict__`.

## TUPLES AS IMMUTABLE LISTS
* tuple supports all list methods that do not involve adding or removing items, with one exception—tuple lacks the `__reversed__` method. However, that is just for optimization; reversed(my_tuple) works without it.

# When a List Is Not the Answer
* If you need to store 10 million floating-point values, an `array` is much more efficient, because an `array` does not actually hold full-fledged **float objects**, but only the packed bytes representing their machine values—just like an array in the C language. 
* On the other hand, if you are constantly adding and removing items from the ends of a `list` as a FIFO or LIFO data structure, a `deque` (double-ended queue) works faster.
* If your code does a lot of containment checks (e.g., `item in my_collection`), consider using a `set` for `my_collection`, especially if it holds a large number of items. Sets are optimized for fast membership checking. But they are not sequences (their content is unordered).

## Arrays

* If the list will only contain numbers, an `array.array` is more efficient than a `list`: it supports all mutable sequence operations (including .pop, .insert, and .extend), and additional methods for fast loading and saving such as .frombytes and .tofile.
  * Another fast and more flexible way of saving numeric data is the pickle module for object serialization. Saving an array of floats with pickle.dump is almost as fast as with array.tofile—however, pickle handles almost all built-in types, including complex numbers, nested collections, and even instances of user-defined classes automatically (if they are not too tricky in their implementation).
* A Python array is as lean as a C array.
* To sort an array
```
a = array.array(a.typecode, sorted(a))
```

## Memoryview
* The built-in memorview class is a shared-memory sequence type that lets you handle slices of arrays without copying bytes.
* A memoryview is essentially a generalized NumPy array structure in Python itself (without the math). It allows you to share memory between data-structures (things like PIL images, SQLlite databases, NumPy arrays, etc.) without first copying. This is very important for large data sets.
* [A short tutorial](https://eli.thegreenplace.net/2011/11/28/less-copies-in-python-with-the-buffer-protocol-and-memoryviews/)

## DEQUES AND OTHER QUEUES
* The .append and .pop methods make a list usable as a stack or a queue (if you use .append and .pop(0), you get LIFO behavior). But inserting and removing from the left of a list (the 0-index end) is costly because the entire list must be shifted.
* The class collections.deque is a thread-safe double-ended queue designed for fast inserting and removing from both ends. 
* There is a hidden cost: removing items from the middle of a deque is not as fast. It is really optimized for appending and popping from the ends.
* The append and popleft operations are atomic, so deque is safe to use as a LIFO queue in multithreaded applications without the need for using locks.





# Common Operations to Python Sequences
## Slicing
### Slice Object

* To evaluate the expression `seq[start:stop:step]`, Python calls `seq.__getitem__(slice(start, stop, step))`.
* The built-in sequence types in Python are one-dimensional, so they support only one index or slice, and not a tuple of them.
* For every sequence in python (except `str`), `s[i]` returns one item, and `s[i:i+1]` returns a sequence of the same type with the `s[1]` item inside it. The only sequence type where `s[0] == s[:1]` is the `str` type.
* [Why numbering should start at zero?](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html)

#### Multidimensional Slice
* Items of a two-dimensional `numpy.ndarray` can be fetched using the syntax `a[i, j]` and a **two-dimensional slice** obtained with an expression like `a[m:n, k:l]`. 
* To evaluate `a[i, j]`, Python calls `a.__getitem__((i, j))`. The `[]` operator simply receive the indices in `a[i, j]` as a tuple.

#### The Ellipsis
* It is an alias to the `Ellipsis` object, the single instance of the `ellipsis` class.
* As such, it can be passed as an argument to functions and as part of a slice specification, as in `f(a, ..., z)` or `a[i:...]`. NumPy uses `...` as a shortcut when slicing arrays of many dimensions; for example, if x is a four-dimensional array, x[i, ...] is a shortcut for x[i, :, :, :,].

### ASSIGNING TO SLICES
* When the target of the assignment is a slice, the right side must be an iterable object, even if it has just one item.
```
>>> l
[0, 1, 20, 11, 5, 22, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
>>> l[2:5] = [100]
>>> l
[0, 1, 100, 22, 9]
```

## Using + and * with Sequences
* Both + and * always create a new object, and never change their operands.
  * Beware of expressions like `a * n` when a is a sequence containing mutable items because the result may surprise you. For example, trying to initialize a list of lists as `my_list = [[]] * 3` will result in a list with three references to the same inner list, which is probably not what you want.
  * Another Example
  ```
  >>> weird_board = [['_'] * 3] * 3 # use this instead: board = [['_'] * 3 for i in range(3)]
  # The outer list is made of three references to the same inner list. While it is unchanged, all seems right.
  >>> weird_board
  [['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
  >>> weird_board[1][2] = 'O'
  >>> weird_board
  [['_', '_', 'O'], ['_', '_', 'O'], ['_', '_', 'O']]
  ```

## Augmented Assignment (`+=` or `*=`) with Sequences
* The augmented assignment operators `+=` and `*=` behave very differently depending on the first operand. 
  * The special method that makes `+=` work is `__iadd__` (for “in-place addition”). However, if `__iadd__` is not implemented, Python falls back to calling `__add__`.
  * Consider `a += b`. In the case of mutable sequences (e.g., `list`, `bytearray`, `array.array`), `a` will be changed in place (i.e., the effect will be similar to `a.extend(b)`). 
  * However, when `a` does not implement `__iadd__`, the expression `a += b` has the same effect as `a = a + b`: the expression `a + b` is evaluated first, producing a new object, which is then bound to `a`. 
  * In other words, the identity of the object bound to a may or may not change, depending on the availability of `__iadd__`.

* In general, for mutable sequences, it is a good bet that `__iadd__` is implemented and that `+=` happens in place. For immutable sequences, clearly there is no way for that to happen.
* Repeated concatenation of immutable sequences is inefficient, because instead of just appending new items, the interpreter has to copy the whole target sequence to create a new one with the new items concatenated.
  * `str` is an exception to this description. Because string building with `+=` in loops is so common in the wild, `CPython` is optimized for this use case. `str` instances are allocated in memory with room to spare, so that concatenation does not require copying the whole string every time.

###  A += Assignment Puzzler
* An odd behavior with Tuples of mutable sequence. The following tuple with `+=` operation both throw error and being modified.
```
>>> t = (1, 2, [30, 40])
>>> t[2] += [50, 60]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> t
(1, 2, [30, 40, 50, 60])
```

* Best Practices
  * Putting mutable items in tuples is not a good idea.
  * Augmented assignment is not an atomic operation—we just saw it throwing an exception after doing part of its job.
  * Inspecting Python bytecode is not too difficult, and is often helpful to see what is going on under the hood.

## `list.sort` and the `sorted` Built-In Function
* list.sort is in-place sort, while `sorted` always returns a new sequence.
* The `key` optional argument of `list.sort`, `sorted`, `max`, and `min` is a great idea. 
  * It is also more efficient because the `key` function is invoked only once per item. Of course, Python also has to compare the keys while sorting, but that comparison is done in optimized C code and not in a Python function that you wrote.



#### Managing Ordered Sequences with bisect
* Use `bisect.insort` does both steps of search and insert for you (i.e. `index = bisect.bisect(haystack, needle)` and `haystack.insert(index, needle)`), and is faster.

#### Timsort
* The sorting algorithm used in sorted and list.sort is Timsort, an adaptive algorithm that switches from insertion sort to merge sort strategies, depending on how ordered the data is.
* Timsort, the sorting algorithm used in Python, is **stable** (i.e., it preserves the relative ordering of items that compare equal). 





