# Table of Contents
* [Functional Programming](#functional-programming)
  * [Hallmarks of Functional Programming](#hallmarks-of-functional-programming)
* [First-Class Objects](#first-class-objects)
* [Functions in Python](#functions-in-python)
  * [Treat Function like an Object](#treat-function-like-an-object)
  * [High Order Functions](#high-order-functions)
  * [Anonymous Functions](#anonymous-functions)
  * [The Seven Flavors of Callable Objects](#the-seven-flavors-of-callable-objects)
* [](#)
* [](#)
* [](#)


* A must read: [Functional Programming Howto](https://docs.python.org/3/howto/functional.html)

# Functional Programming

## Hallmarks of Functional Programming

* First-Class Functions
* Higher Order Functions
* Immutable Data
* Pure Functions
* Recursion
* Manipulation of Lists
* Lazy Evaluation

# First-Class Objects

* “first-class object” is defined as a program entity that can be:
  * Created at runtime
  * Assigned to a variable or element in a data structure
  * Passed as an argument to a function
  * Returned as the result of a function

* Python 
  * Functions are first-class objects
  * Integers, strings, and dictionaries are first-class objects in Python too

* C++
  * Non First-Class Objects in C++
    * Functions and class methods are NOT first-class objects.
    * Classes are NOT first class in C++.
  * First-Class Objects
    * Polymorphic function wrappers are first-class objects
      * `std::function` are first-class objects in C++. e.g. `std::function<double(double,double)>` can be passed around
    * Lambda functions are first-class objects since C++ 11.
    * All objects in C++ are first-class objects.
      * Function objects are first-class objects.
    * Function pointers (even C has this) are first-class objects.
    * Coroutines may come in C++20.
    * First-class functions are heavily used in Standard Template Library.
      * `std::accumulate` is a higher order function and can accept first-class functions

# Functions in Python

## Functions Treated Like an Object
* Function object itself is an instance of the `function` class.

## High Order Functions
> A function that takes a function as argument or returns a function as the result is a higher-order function.

### Reducing Functions
* `map` and `filter`: A `listcomp` or a `genexp` does the job of `map` and `filter` combined, but is more readable.
  * In Python 3, `map` and `filter` return `generators`—a form of `iterator`—so their direct substitute is now a `generator expression` (in Python 2, these functions returned lists, therefore their closest alternative is a `listcomp`).
* `reduce`: It was demoted from a built-in in Python 2 to the `functools` module in Python 3.
  * Its most common use case, `summation`, is better served by the `sum` built-in. This is a big win in terms of readability and performance.
* `apply`: removed in Python 3  
* Other reducing built-ins are `all` and `any`.
  * `all([])` returns True
  * `any([])` returns False

### Other High Order Functions
* The `sorted`, `min`, `max` built-ins, and `functools.partial` functions.

## Anonymous Functions
* The simple syntax of Python limits the body of `lambda` functions to be pure expressions. In other words, the body of a lambda cannot make assignments or use any other Python statement such as while, try, etc.

## The Seven Flavors of Callable Objects

* The call operator (i.e., `()`) may be applied to other objects beyond user-defined functions. To determine whether an object is callable, use the `callable()` built-in function. The Python Data Model documentation lists seven callable types:
  * User-defined functions: Created with `def` statements or `lambda` expressions.
  * Built-in functions: A function implemented in C (for CPython), like `len` or `time.strftime`.
  * Built-in methods: Methods implemented in C, like `dict.get`.
  * Methods: Functions defined in the body of a class.
  * Classes: When invoked, a class runs its `__new__` method to create an instance, then `__init__` to initialize it, and finally the instance is returned to the caller. Because there is no new operator in Python, calling a class is like calling a function. (Usually calling a class creates an instance of the same class, but other behaviors are possible by overriding `__new__`.)
  * Class instances: If a class defines a `__call__` method, then its instances may be invoked as functions. They are "User-Defined Callable Types".
    * * An example is a `decorator`. Decorators must be functions, but it is sometimes convenient to be able to “remember” something between calls of the decorator (e.g., for memoization—caching the results of expensive computations for later use).
    * A totally different approach to creating functions with internal state is to use `closures`.
  * Generator functions: Functions or methods that use the `yield` keyword. When called, generator functions return a generator object.

## Function Introspection
* Like the instances of a plain user-defined class, a function uses the `__dict__` attribute to store user attributes assigned to it.
  * `Django` is the framework that uses it. For example, the `short_description`, `boolean`, and `allow_tags` attributes.

## Function Parameters
### Use of `*` and `**` to “explode” iterables and mappings into separate arguments when we call a function
* Prefixing the `my_tag` dict with `**` passes all its items as separate arguments, which are then bound to the named parameters, with the remaining caught by `**attrs` as below:
```
def tag(name, *content, cls=None, **attrs):
    ...
    
my_tag = {'name': 'img', 'title': 'Sunset Boulevard', 'src': 'sunset.jpg', 'cls': 'framed'}
tag(**my_tag) 
'<img class="framed" src="sunset.jpg" title="Sunset Boulevard" />'    
```

### Keyword-only arguments
* The `cls` parameter above can only be given as a keyword argument—it will never capture unnamed positional arguments. 
* To specify keyword-only arguments when defining a function, name them after the argument prefixed with `*`. 
* If you don’t want to support variable positional arguments but still want keyword-only arguments, put a `*` by itself in the signature, like this:
```
>>> def f(a, *, b):
...     return a, b
...
>>> f(1, b=2)
(1, 2)
```
* keyword-only arguments do not need to have a default value: they can be mandatory, like `b` in the preceding example.

### Retrieving Information About Parameters







  
