
# Table of Contents
* [Generic Mapping Types](#generic-mapping-types)
* [Python Dicts](#python-dicts)
  * [Handling Missing Keys](#handling-missing-keys)
  * [The `view` object in `dict`](#)
  * [ChainedMap](#chainedmap)
  * [UserDict](#userdict)
  * [Immutable Mappings](#immutable-mappings)
* [Python Sets](#python-sets)
  * [Set Literals](#set-literals)
  * [Set Comprehensions (setcomps)](#)
* [Hash Table](#hash-table)
  * [Hash Buckets](#hash-buckets)
  * [Hash Function](#hash-function)
  * [The Hash Table Algorithm](#the-hash-table-algorithm)
  * [Practical Consequences of Hash Table on Dicts and Sets](#practical-consequences-of-hash-table-on-dicts-and-sets)
* [References](#references)


# Generic Mapping Types
* The `collections.abc` module provides the `Mapping` and `MutableMapping` `ABC`s to formalize the interfaces of dict and similar types.
* All mapping types in the standard library use the basic dict in their implementation, so they share the limitation that the keys must be hashable (the values need not be hashable, only the keys).

# Python Dicts
* Python dicts are highly optimized. Hash tables are the engines behind Python’s high-performance dicts.
* The built-in functions live in `__builtins__.__dict__`.


### An Example of Duck Typing
* The way `update` handles its first argument `m` is a prime example of `duck typing`.
  * It first checks whether `m` has a keys method and, if it does, assumes it is a mapping. 
  * Otherwise, `update` falls back to iterating over `m`, assuming its items are (key, value) pairs. 
* The constructor for most Python mappings uses the logic of `update` internally, which means they can be initialized from other mappings or from any iterable object producing (key, value) pairs.

## Handling Missing Keys
#### HANDLING MISSING KEYS WITH `setdefault`
* In line with the **fail-fast** philosophy, `dict` access with `d[k]` raises an error when `k` is not an existing key.
* However, `dict.get` is not the best way to handle a missing key when updating the value found (if it is mutable).
```
my_dict.setdefault(key, []).append(new_value) #This is better
```

The following code should be replaced with above one liner. It performs at least two searches for key while `setdefault` does it all with a single lookup.

```
occurrences = my_dict.get(key, []) # one search
occurrences.append(new_value)    # 
my_dict[word] = occurrences      # Another search through my_dict
```

#### `defaultdict`: ANOTHER TAKE ON MISSING KEYS
* The mechanism that makes `defaultdict` work by calling `default_factory` is actually the `__missing__` special method, a feature supported by all standard mapping types.

```
index = collections.defaultdict(list)
index[word].append(location)
```

#### Overwrite the `__missing__` Method
* If you subclass `dict` and provide a `__missing__` method, the standard `dict.__getitem__` will call it whenever a key is not found, instead of raising `KeyError`.
* The `__missing__` method is just called by `__getitem__` (i.e., for the `d[k]` operator). The presence of a `__missing__` method has no effect on the behavior of other methods that look up keys, such as get or `__contains__` (which implements the in operator). This is why the default_factory of defaultdict works only with `__getitem__`.


## The `view` object in `dict`
* A search like `k in my_dict.keys()` is efficient in Python 3 even for very large mappings because `dict.keys()` returns a `view` (so are `dict.values()` and `dict.items()`), which is similar to a `set`, and containment checks in `sets` are as fast as in dictionaries.
* They behave more like sets than the lists returned by these methods in Python 2. 
* They provide a dynamic view on the dictionary’s entries, they do not replicate the contents of the dict, and they immediately reflect any changes to the dict.
* If all values are hashable, so that `(key, value)` pairs are unique and hashable, then the `items` view is also set-like. (`Values` views are not treated as set-like since the entries are generally not unique.) 

## ChainedMap
* A ChainMap groups multiple dicts or other mappings together to create a single, updateable view. It holds a list of mappings that can be searched as one. The lookup is performed on each mapping in order, and succeeds if the key is found in any of them. 
* This is useful to interpreters for languages with nested scopes, where each mapping represents a scope context.
```
import builtins
pylookup = ChainMap(locals(), globals(), vars(builtins))
```
## UserDict
* `UserDict` is designed to be subclassed. A better way to create a user-defined mapping type is to subclass `collections.UserDict` instead of `dict`.
* The main reason why it’s preferable to subclass from UserDict rather than from dict is that the built-in has some implementation shortcuts that end up forcing us to override methods that we can just inherit from UserDict with no problems.

## Immutable Mappings
* The mapping types provided by the standard library are all mutable, but you may need to guarantee that a user cannot change a mapping by mistake.
* Since Python 3.3, the `types` module provides a wrapper class called MappingProxyType, which, given a mapping, returns a mappingproxy instance that is a read-only but dynamic view of the original mapping. This means that updates to the original mapping can be seen in the mappingproxy, but changes cannot be made through it. 


# Python Set
* Set elements must be hashable. The `set` type is not hashable, but `frozenset` is, so you can have `frozenset` elements inside a `set`.

## Set Literals
* There’s no literal notation for the empty set, so we must remember to write set(). `{}` is a dict.
* Literal set syntax like `{1, 2, 3}` is both faster and more readable than calling the constructor (e.g., `set([1, 2, 3])`). 
  * The latter form is slower because, to evaluate it, Python has to look up the set name to fetch the constructor, then build a list, and finally pass it to the constructor. 
  * In contrast, to process a literal like `{1, 2, 3}`, Python runs a specialized `BUILD_SET` bytecode.
  
## Set Comprehensions (setcomps)
```
{chr(i) for i in range(32, 256)}
```

## Hash Table
### Hash Buckets
* A hash table is a sparse array (i.e., an array that always has empty cells). In standard data structure texts, the cells in a hash table are often called “buckets.” 
* In a dict hash table, there is a bucket for each item, and it contains two fields: a reference to the key and a reference to the value of the item. 
* Because all buckets have the same size, access to an individual bucket is done by offset.
* Python tries to keep at least 1/3 of the buckets empty; if the hash table becomes too crowded, it is copied to a new location with room for more buckets.

### Hash Function
* If two objects compare equal, their hash values must also be equal, otherwise the hash table algorithm does not work. 
  * A CPython implementation detail: the hash value of an `int` that fits in a machine word is the value of the int itself.
  * Example: `hash(1) == hash(1.0)`
  
* To be effective as hash table indexes, hash values should scatter around the index space as much as possible. This means that, ideally, objects that are similar but not equal should have hash values that differ widely.
* Starting with Python 3.3, a random **salt** value is added to the hashes of `str`, `bytes`, and `datetime` objects. The salt value is constant within a Python process but varies between interpreter runs. The random salt is a security measure to prevent a DoS attack.
  * This is intended to provide protection against a **denial-of-service*** (DoS) caused by carefully-chosen inputs that exploit the predictable collisions in the underlying hashing algorithms (the worst case performance of a dict insertion, O(n^2) complexity, really???  I thought it's O(n)). See http://www.ocert.org/advisories/ocert-2011-003.html for details.
    * The attacker, using specially crafted HTTP requests, can lead to a 100% of CPU usage which can last up to several hours depending on the targeted application and server performance, the amplification effect is considerable and requires little bandwidth and time on the attacker side.
  * Changing hash values affects the iteration order of sets. Python has never made guarantees about this ordering (and it typically varies between 32-bit and 64-bit builds).

### The Hash Table Algorithm

* To fetch the value at `my_dict[search_key]`, Python calls `hash(search_key)` to obtain the hash value of `search_key` and uses **the least significant bits** of that number as an offset to look up a bucket in the hash table (the number of bits used depends on the current size of the table). 
  * If the found bucket is empty, `KeyError` is raised. 
  * Otherwise, the found bucket has an item—a `found_key:found_value` pair—and then Python checks whether `search_key == found_key`. 
    * If they match, that was the item sought: `found_value` is returned.
    * However, if `search_key` and `found_key` do not match, this is a `hash collision`. 
    
* `Hash Collision` happens because a hash function maps arbitrary objects to a small number of bits, and—in addition—the hash table is indexed with a subset of those bits. 
  * In order to resolve the collision, the algorithm then takes different bits in the hash, massages them in a particular way, and uses the result as an offset to look up a different bucket. If that is empty, `KeyError` is raised; if not, either the keys match and the item value is returned, or ***the collision resolution process is repeated**. 
  
* Additionally, when **inserting** items, Python may determine that the hash table is too crowded and rebuild it to a new location with more room. As the hash table grows, so does the number of hash bits used as bucket offsets, and this keeps the rate of collisions low.

* This implementation may seem like a lot of work, but even with millions of items in a dict, many searches happen with no collisions, and the average number of collisions per search is between one and two. Under normal usage, even the unluckiest keys can be found after a handful of collisions are resolved.

### Practical Consequences of Hash Table on Dicts and Sets

* The set and frozenset types are also implemented with a hash table, except that each bucket holds only a reference to the element (as if it were a key in a dict, but without a value to go with it). 
* Every practical consequence said here applies to a set as well.

#### Keys must be hashable objects
* An object is hashable if all of these requirements are met:
  * It supports the `hash()` function via a `__hash__()` method that always returns the same value over the lifetime of the object.
  * It supports equality comparison via an `eq()` method (`__eq__()`).
  * If a == b is True then hash(a) == hash(b) must also be True.
  
### Hashable Objects
* Hashable objects which compare equal must have the same hash value.  
* The atomic immutable types (`str`, `bytes`, numeric types) are all hashable.
* A `frozenset` is always hashable, because its elements must be hashable by definition. 
* A `tuple` is hashable only if all its items are hashable.
* User-defined types are hashable by default because their hash value is their id() and they all compare not equal.
  * If you implement a class with a custom `__eq__` method, you must also implement a suitable `__hash__`, because you must always make sure that if `a == b` is True then `hash(a) == hash(b)` is also True. Otherwise you are breaking an invariant of the hash table algorithm, with the grave consequence that dicts and sets will not deal reliably with your objects. 
  * If a custom `__eq__` depends on mutable state, then `__hash__` must raise `TypeError` with a message like unhashable type: 'MyClass'. That is it may be hashable only if all its attributes are immutable.

#### dicts have significant memory overhead
* If you are handling a large quantity of records, it makes sense to store them in a list of tuples or named tuples instead of using a list of dictionaries in JSON style, with one dict per record. Replacing dicts with tuples reduces the memory usage in two ways: 
  * by removing the overhead of one hash table per record
  * by not storing the field names again with each record.
* For user-defined types, the `__slots__` class attribute changes the storage of instance attributes from a dict to a tuple in each instance. 
* Keep in mind we are talking about space optimizations. If you are dealing with a few million objects and your machine has gigabytes of RAM, you should postpone such optimizations until they are actually warranted. Optimization is the altar where maintainability is sacrificed.

#### Key search is very fast
* Dictionaries have significant memory overhead, but they provide fast access regardless of the size of the dictionary—as long as it fits in memory.

#### Key ordering depends on insertion order
* When a hash collision happens, the second key ends up in a position that it would not normally occupy if it had been inserted first. So, a `dict` built as `dict([(key1, value1), (key2, value2)])` compares equal to `dict([(key2, value2), (key1, value1)])`, but their key ordering may not be the same if the hashes of `key1` and `key2` collide.

#### Adding items to a dict may change the order of existing keys
* Whenever you add a new item to a dict, the Python interpreter may decide that the hash table of that dictionary needs to grow. This entails building a new, bigger hash table, and adding all current items to the new table. During this process, new (but different) hash collisions may happen, with the result that the keys are likely to be ordered differently in the new hash table. 
  * All of this is implementation-dependent, so you cannot reliably predict when it will happen. 
  * If you are iterating over the dictionary keys and changing them at the same time, your loop may not scan all the items as expected—not even the items that were already in the dictionary before you added to it.

* This is why modifying the contents of a dict while iterating through it is a bad idea. If you need to scan and add items to a dictionary, do it in two steps: read the dict from start to finish and collect the needed additions in a second dict. Then update the first one with it.

## References
1. Written by A.M. Kuchling—a Python core contributor and author of many pages of the official Python docs and how-tos—Chapter 18, “Python’s Dictionary Implementation: Being All Things to All People, in the book Beautiful Code (O’Reilly) includes a detailed explanation of the inner workings of the Python dict. 
1. There are lots of comments in the source code of the [dictobject.cCPython module](https://hg.python.org/cpython/file/tip/Objects/dictobject.c). 
1. Brandon Craig Rhodes’ presentation [The Mighty Dictionary](https://pyvideo.org/pycon-us-2010/the-mighty-dictionary-55.html) is excellent and shows how hash tables work by using lots of slides with… tables!




















