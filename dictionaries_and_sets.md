
# Table of Contents
* [Generic Mapping Types](#generic-mapping-types)
* [Python Dicts](#python-dicts)


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

### HANDLING MISSING KEYS WITH SETDEFAULT
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

