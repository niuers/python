
# Table of Contents
* [Generic Mapping Types](#generic-mapping-types)
  * [Hashable](#hashable)
* [Python Dicts](#python-dicts)
  * [Handling Missing Keys](#handling-missing-keys)
* [Python Sets](#python-sets)


# Generic Mapping Types
* The `collections.abc` module provides the `Mapping` and `MutableMapping` `ABC`s to formalize the interfaces of dict and similar types.
* All mapping types in the standard library use the basic dict in their implementation, so they share the limitation that the keys must be hashable (the values need not be hashable, only the keys).

### Hashable
* An object is hashable if it has a hash value which never changes during its lifetime (it needs a `__hash__()` method), and can be compared to other objects (it needs an `__eq__()` method). 
* Hashable objects which compare equal must have the same hash value.
  * The atomic immutable types (`str`, `bytes`, numeric types) are all hashable. 
  * A `frozenset` is always hashable, because its elements must be hashable by definition. 
  * A `tuple` is hashable only if all its items are hashable.
  * User-defined types are hashable by default because their hash value is their `id()` and they all compare not equal. If an object implements a custom `__eq__` that takes into account its internal state, it may be hashable only if all its attributes are immutable.
  
# Python Dicts
* Python dicts are highly optimized. Hash tables are the engines behind Python’s high-performance dicts.
* The built-in functions live in __builtins__.__dict__.


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
* They provide a dynamic view on the dictionary’s entries, which means that when the dictionary changes, the view reflects these changes.
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
