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

## Iterables versus Iterators
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

* A common cause of errors in building iterables and iterators is to confuse the two. To be clear: iterables have an __iter__ method that instantiates a new iterator every time. Iterators implement a __next__ method that returns individual items, and an __iter__ method that returns self. Therefore, iterators are also iterable, but iterables are not iterators.
* An iterable should never act as an iterator over itself. In other words, iterables must implement __iter__, but not __next__.

## Generator Function
* Any Python function that has the yield keyword in its body is a generator function: a function which, when called, returns a generator object. In other words, a generator function is a generator factory.
* Generators are iterators that produce the values of the expressions passed to yield.
* A generator function builds a generator object that wraps the body of the function. When we invoke next(…) on the generator object, execution advances to the next yield in the function body, and the next(…) call evaluates to the value yielded when the function body is suspended. Finally, when the function body returns, the enclosing generator object raises StopIteration, in accordance with the Iterator protocol.
  * A generator function which, when called, builds a generator object that implements the iterator interface. 

* A generator yields or produces values. A generator doesn’t “return” values in the usual way: the return statement in the body of a generator function causes StopIteration to be raised by the generator object.

* Example with `for-loop`
  * To iterate, the for machinery does the equivalent of `g = iter(gen_AB())` to get a generator object, and then `next(g)` (before the `for` boday) at each iteration.
  ```
  for c in gen_AB():
      print('-->', c)
  ```
  * When the generator function body runs to the end, the generator object raises StopIteration. The for loop machinery catches that exception, and the loop terminates cleanly.

### Lazy Implementation
* Whenever you are using Python 3 and start wondering “Is there a lazy way of doing this?”, often the answer is “Yes.”

### Generator Expression
* A generator expression can be understood as a lazy version of a list comprehension: it does not eagerly build a list, but returns a generator that will lazily produce the items on demand. In other words, if a list comprehension is a factory of lists, a generator expression is a factory of generators.
* Generator expressions are syntactic sugar: they can always be replaced by generator functions, but sometimes are more convenient.
* On the other hand, generator functions are much more flexible: you can code complex logic with multiple statements, and can even use them as coroutines.
* My rule of thumb in choosing the syntax to use is simple: if the generator expression spans more than a couple of lines, I prefer to code a generator function for the sake of readability. Also, because generator functions have a name, they can be reused. You can always name a generator expression and use it later by assigning it to a variable, of course, but that is stretching its intended usage as a one-off generator.
* When a generator expression is passed as the single argument to a function or constructor, you don’t need to write a set of parentheses for the function call and another to enclose the generator expression. A single pair will do, like the example below. However, if there are more function arguments after the generator expression, you need to enclose it in parentheses to avoid a SyntaxError:

```
def __mul__(self, scalar):
    if isinstance(scalar, numbers.Real):
        return Vector(n * scalar for n in self)
    else:
        return NotImplemented
```







