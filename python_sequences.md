
# Classifications
### Container Sequences vs. Flat Sequences
* Container Sequences
> list, tuple, and collections.deque can hold items of different types. Container sequences hold references to the objects they contain, which may be of any type.

* Flat Sequences
> str, bytes, bytearray, memoryview, and array.array hold items of one type. Flat sequences physically store the value of each item within its own memory space, and not as distinct objects. Thus, flat sequences are more compact, but they are limited to holding primitive values like characters, bytes, and numbers.

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
### Tuples as records
* Tuples hold records: each item in the tuple holds the data for one field and the position of the item gives its meaning.
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
* Instances of a class that you build with namedtuple take exactly the same amount of memory as tuples because the field names are stored in the class. They use less memory than a regular object because they don’t store attributes in a per-instance __dict__.

### TUPLES AS IMMUTABLE LISTS
* tuple supports all list methods that do not involve adding or removing items, with one exception—tuple lacks the __reversed__ method. However, that is just for optimization; reversed(my_tuple) works without it.




