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


# Functional Programming
* A must read: [Functional Programming Howto](https://docs.python.org/3/howto/functional.html)

## Programming Paradigms

* Procedural programs are lists of instructions that tell the computer what to do with the program’s input. 
  * C, Pascal, and even Unix shells are procedural languages.
  
* Declarative languages: you write a specification that describes the problem to be solved, and the language implementation figures out how to perform the computation efficiently. 
  * SQL is the declarative language: a SQL query describes the data set you want to retrieve, and the SQL engine decides whether to scan tables or use indexes, which subclauses should be performed first, etc.
  
* Object-oriented programs manipulate collections of objects. 
  * Objects have internal state and support methods that query or modify this internal state in some way. 
  * Smalltalk and Java are object-oriented languages. 
  * C++ and Python are languages that support object-oriented programming, but don’t force the use of object-oriented features.

* Functional programming decomposes a problem into a set of functions. 
  * Ideally, functions only take inputs and produce outputs, and don’t have any internal state that affects the output produced for a given input.
  * Well-known functional languages include the ML family (Standard ML, OCaml, and other variants) and Haskell.
  * Python is, by design, *NOT* a functional language—whatever that means. Python just borrows a few good ideas from functional languages.

## Theoretical and Practical Advantages to the Functional Style
### Formal provability
* A theoretical benefit is that it’s easier to construct a mathematical proof that a functional program is correct.
* Functional programming’s avoidance of assignments arose because assignments are difficult to handle with this technique; assignments can break invariants that were true before the assignment without producing any new invariants that can be propagated onward.

### Modularity
* It forces you to break apart your problem into small pieces.

### Composability
* Often you’ll assemble new programs by arranging existing functions in a new configuration and writing a few functions specialized for the current task.

### Ease of debugging and testing
* Debugging is simplified because functions are generally small and clearly specified.

## Hallmarks of Functional Programming

* First-Class Functions
* Higher Order Functions
* Immutable Data
* Pure Functions
  * Pure functions are functions that have no side effects.
  * Functional programming tries to avoid side effects, meaning not using data structures that get updated as a program runs; every function’s output must only depend on its input.

* Recursion
* Manipulation of Lists
* Lazy Evaluation

## First-Class Objects

* “first-class object” is defined as a program entity that can be:
  * Created at runtime
  * Assigned to a variable or element in a data structure
  * Passed as an argument to a function
  * Returned as the result of a function

* Python 
  * Functions are first-class objects
  * Modules in Python are also first-class objects, and the standard library provides several functions to handle them.
    * globals(): Return a dictionary representing the current global symbol table. This is always the dictionary of the current module (inside a function or method, this is the module where it is defined, not the module from which it is called).
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

## Python Features Important to Functional Programming
### Iterators
* An iterator is an object representing a stream of data; this object returns the data one element at a time.
* Note that you can only go forward in an iterator; there’s no way to get the previous element, reset the iterator, or make a copy of it.
* List comprehensions (return list) and generator expressions (return iterator) are common operations.
  * Generator expressions return an iterator that computes the values as necessary, not needing to materialize all the values at once. This means that list comprehensions aren’t useful if you’re working with iterators that return an infinite stream or a very large amount of data. Generator expressions are preferable in these situations.
  * Generator expressions always have to be written inside parentheses, but the parentheses signalling a function call also count.

### Generators
* Generators are a special class of functions that simplify the task of writing iterators. Regular functions compute a value and return it, but generators return an iterator that returns a stream of values.
* The big difference between yield and a return statement is that on reaching a yield the generator’s state of execution is suspended and local variables are preserved. On the next call to the generator’s `__next__()` method, the function will resume executing.



# Functions in Python

## Functions Treated Like an Object
* Function object itself is an instance of the `function` class.

## High Order Functions
> A function that takes a function as argument or returns a function as the result is a higher-order function.

### Reducing Functions
* `map` and `filter`: A `listcomp` (coming from Haskell) or a `genexp` does the job of `map` and `filter` combined, but is more readable.
  * In Python 3, `map` and `filter` return `generators`—a form of `iterator`—so their direct substitute is now a `generator expression` (in Python 2, these functions returned lists, therefore their closest alternative is a `listcomp`).
* `reduce`: It was demoted from a built-in in Python 2 to the `functools` module in Python 3.
  * Its most common use case, `summation`, is better served by the `sum` built-in. This is a big win in terms of readability and performance.
* `apply`: removed in Python 3  
* Other reducing built-ins are `all` and `any`.
  * `all([])` returns True
  * `any([])` returns False

### Other High Order Functions
* The `sorted`, `min`, `max` built-ins, and `functools.partial` functions.
* Function decorators

## Anonymous Functions
* The simple syntax of Python limits the body of `lambda` functions to be pure expressions. In other words, the body of a lambda cannot make assignments or use any other Python statement such as while, try, etc.
* They don't have name, this makes them hard to debug.
* Asynchronous programming in Python is more structured, perhaps because the limited lambda demands it. 

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
* Using the attributes on function objects is awkward
  * Within a function object, the` __defaults__` attribute holds a tuple with the default values of positional and keyword arguments. 
  * The defaults for keyword-only arguments appear in `__kwdefaults__`. The names of the arguments, however, are found within the `__code__` attribute, which is a reference to a code object with many attributes of its own.
  
* Use `inspect` module
```
>>> from inspect import signature
>>> sig = signature(clip)
```

### Function Annotations
* Python 3 provides syntax to attach metadata to the parameters of a function declaration and its return value.
* No processing (no checks, enforcement, validation, or any other action is performed) is done with the annotations. They are merely stored in the `__annotations__` attribute of the function, a `dict`. In other words, annotations have no meaning to the Python interpreter.
* The biggest impact of function annotations will probably not be dynamic settings such as Bobo, but in providing optional type information for static type checking in tools like IDEs and linters.


## Packages for Functional Programming
### THE `operator` MODULE
* To save you the trouble of writing trivial anonymous functions like lambda a, b: a*b, the operator module provides function equivalents for dozens of arithmetic operators. e.g. `mul`
* Another group of one-trick lambdas that operator replaces are functions to pick items from sequences or read attributes from objects: `itemgetter` and `attrgetter` actually build custom functions to do that.
  * Because `itemgetter` uses the `[]` operator, it supports not only sequences but also mappings and any class that implements `__getitem__`.
  
### The `functools` Module
* Freezing arguments with `functools.partial`
* An impressive functools function is `lru_cache`, which does memoization—a form of automatic optimization that works by storing the results of function calls to avoid expensive recalculations.

## Design Patterns with Functions
### General Idea
* Although design patterns are language-independent, that does not mean every pattern applies to every language. 
* In his 1996 presentation, “Design Patterns in Dynamic Languages”, Peter Norvig states that 16 out of the 23 patterns in the original Design Patterns book by Gamma et al. become either “invisible or simpler” in a dynamic language (slide 9).
  * The message from Peter Norvig’s design patterns slides is that the Command and Strategy patterns—along with Template Method and Visitor—can be made simpler or even “invisible” with first-class functions, at least for some applications of these patterns.
* The general idea is: you can replace instances of some participant class in these patterns with simple functions, reducing a lot of boilerplate code.

### Strategy Pattern
> It defines a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

#### Participants
* Context
  * Provides a service by delegating some computation to interchangeable components that implement alternative algorithms. 
  * In the ecommerce example, the context is an Order, which is configured to apply a promotional discount according to one of several algorithms.
* Strategy
  * The interface common to the components that implement the different algorithms. 
  * In our example, this role is played by an abstract class called Promotion (abstract class with Promotion() method)
* Concrete Strategy
  * One of the concrete subclasses of Strategy. 
  * FidelityPromo, BulkPromo, and LargeOrderPromo are the three concrete strategies implemented with Promotion() methods.

#### Refactoring with Functions
* When there's no internal state in Strategy classes (so the Strategy classes only deal with data from the context), we can replace the concrete strategies with simple functions (using Context as input) and remove the *Strategy* abstract class.
* With complex concrete strategies holding internal state, it may require all the pieces of the Strategy and Flyweight design patterns combined. 
  * Closures can also be used to store states between function calls.


#### Refactoring with Modules
* You can also refactor with modules using `global()` to implement a 'meta-strategy' that can dynamically choose the best promotion.
```
promos = [globals()[name] for name in globals()
            if name.endswith('_promo')
            and name != 'best_promo'] 
```

* Use `inspect` module
  * The function inspect.getmembers returns the attributes of an object—in this case, the promotions module—optionally filtered by a predicate (a boolean function). We use inspect.isfunction to get only the functions from the module.

```
promos = [func for name, func in
                inspect.getmembers(promotions, inspect.isfunction)]
```

#### Use Decorator
* A more explicit alternative for dynamically collecting promotional discount functions would be to use a simple decorator.

### Command Pattern
* The goal of Command is to decouple an object that invokes an operation (the Invoker) from the provider object that implements it (the Receiver). 
  * In the example from Design Patterns, each invoker is a menu item in a graphical application, and the receivers are the document being edited or the application itself.
* The idea is to put a Command object between the two, implementing an interface with a single method, `execute`, which calls some method in the Receiver to perform the desired operation. 
  * That way the Invoker does not need to know the interface of the Receiver, and different receivers can be adapted through different Command subclasses. 
  * The Invoker is configured with a concrete command and calls its execute method to operate it.
  * You can have a `MacroCommand` that stores a sequence of commands; its execute() method calls the same method in each command stored.
* Quoting from Gamma et al., “Commands are an object-oriented replacement for callbacks.” The question is: do we need an object-oriented replacement for callbacks? Sometimes yes, but not always.

#### Refactoring with Functions
* Instead of giving the Invoker a Command instance, we can simply give it a function. 
* Instead of calling command.execute(), the Invoker can just call command(). 
  * The `MacroCommand` can be implemented with a class implementing `__call__`. Instances of MacroCommand would be callables, each holding a list of functions for future invocation. 

* In many cases, functions or callable objects provide a more natural way of implementing callbacks in Python than mimicking the Strategy or the Command patterns. 

### Visitor Pattern
* “Recipe 8.21. Implementing the Visitor Pattern,” in the Python Cookbook, Third Edition (O’Reilly), by David Beazley and Brian K. Jones, presents an elegant implementation of the Visitor pattern in which a NodeVisitor class handles methods as first-class objects.








  
