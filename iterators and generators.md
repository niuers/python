# Table of Contents

*[Iterator Pattern](#iterator-pattern)
*[](#)
*[](#)
*[](#)



# Iterator Pattern
* Iteration is fundamental to data processing. And when scanning datasets that don’t fit in memory, we need a way to fetch the items lazily, that is, one at a time and on demand. This is what the Iterator pattern is about.
* The yield keyword allows the construction of generators, which work as iterators.
* Every generator is an iterator: generators fully implement the iterator interface. But an iterator—as defined in the GoF book—retrieves items from a collection, while a generator can produce items “out of thin air.” That’s why the Fibonacci sequence generator is a common example: an infinite series of numbers cannot be stored in a collection. However, be aware that the Python community treats iterator and generator as synonyms most of the time.

* Every collection in Python is iterable, and iterators are used internally to support:
  * for loops
  * Collection types construction and extension
  * Looping over text files line by line
  * List, dict, and set comprehensions
  * Tuple unpacking
  * Unpacking actual parameters with * in function calls

# The `iter` function
## How does Iter function Work?
* Whenever the interpreter needs to iterate over an object `x`, it automatically calls `iter(x)`. The iter built-in function:
  * Checks whether the object implements `__iter__`, and calls that to obtain an iterator.
  * If `__iter__` is not implemented, but `__getitem__` is implemented, Python creates an iterator that attempts to fetch items in order, starting from index 0 (zero). The special handling of `__getitem__` exists for backward compatibility reasons and may be gone in the future
  * If that fails, Python raises TypeError, usually saying “C object is not iterable,” where C is the class of the target object.

* This is an extreme form of duck typing: an object is considered iterable not only when it implements the special method `__iter__`, but also when it implements `__getitem__`, as long as `__getitem__` accepts int keys starting from 0.

* In the goose-typing approach, the definition for an iterable is simpler but not as flexible: an object is considered iterable if it implements the __iter__ method. No subclassing or registration is required, because abc.Iterable implements the __subclasshook__

* The most accurate way to check whether an object x is iterable is to call iter(x) and handle a TypeError exception if it isn’t. This is more accurate than using isinstance(x, abc.Iterable), because iter(x) also considers the legacy __getitem__ method, while the Iterable ABC does not.

## Iterables Versus Iterators
### Iterable
* Any object from which the iter built-in function can obtain an iterator. Objects implementing an `__iter__` method returning an iterator are iterable. Sequences are always iterable; as are objects implementing a `__getitem__` method that takes 0-based indexes.

### Iterator
> iterator: Any object that implements the `__next__` no-argument method that returns the next item in a series or raises StopIteration when there are no more items. Python iterators also implement the `__iter__` method so they are iterable as well.

* It’s important to be clear about the relationship between iterables and iterators: Python obtains iterators from iterables.
* The standard interface for an iterator has two methods:
  * `__next__`: Returns the next available item, raising StopIteration when there are no more items.
  * `__iter__`: Returns self; this allows iterators to be used where an iterable is expected, for example, in a for loop.
* This is formalized in the `collections.abc.Iterator` ABC, which defines the `__next__` abstract method, and subclasses Iterable—where the abstract `__iter__` method is defined.

* Iterators in Python aren't a matter of type but of protocol.  A large and changing number of builtin types implement *some* flavor of
iterator.  Don't check the type!  Use hasattr to check for both "__iter__" and "__next__" attributes instead.

* Taking into account the advice from "Lib/types.py" and the logic implemented in "Lib/_collections_abc.py", the best way to check if an object `x` is an iterator is to call `isinstance(x, abc.Iterator)`. Thanks to `Iterator.__subclasshook__`, this test works even if the class of `x` is not a real or virtual subclass of Iterator.





