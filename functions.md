# Table of Contents
* [Functional Programming](#functional-programming)
  * [Programming Paradigms](#programming-paradigms)
  * [Theoretical and Practical Advantages to the Functional Style](#theoretical-and-practical-advantages-to-the-functional-style)
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

# Decorators and Closures

* A decorator is a callable that takes another function as argument (the decorated function). The decorator may perform some processing with the decorated function, and returns it or replaces it with another function or callable object.
* Code that uses inner functions almost always depends on closures to operate correctly.
* Decorators can be stacked.

## When Python Executes Decorators
* They are executed immediately when a module is loaded. That is they run right after the decorated function is defined. That is usually at import time (i.e., when a module is loaded by Python).
* But the decorated functions only run when they are explicitly invoked. This highlights the difference between what Pythonistas call **import time** and **runtime**.

### Decorator-Enhanced Strategy Pattern
* One can use decorator to register `Promotion` functions in previous Strategy Pattern example.

## Variable Scope Rules
* Python does not require you to declare variables, but assumes that a variable **assigned** a value in the body of a function is local. 
```
>>> b = 6
>>> def f2(a):
...     print(a)
...     print(b)
...     b = 9
...
>>> f2(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in f2
UnboundLocalError: local variable 'b' referenced before assignment
```
  * This is much better than the behavior of JavaScript, which does not require variable declarations either, but if you do forget to declare that a variable is local (with var), you may clobber a global variable without knowing.
* If a variable is referrenced without any assignment in the function body, it's treated as a global variable.
```
>>> def f1(a):
...     print(a)
...     print(b)
...
>>> f1(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in f1
NameError: global name 'b' is not defined
```
* If we want the interpreter to treat a variable as a *global variable* in spite of the assignment within the function, we use the global declaration:
```
>>> b = 6
>>> def f3(a):
...     global b
...     print(a)
...     print(b)
...     b = 9
...
>>> f3(3)
3
6
>>> b
9

a = 3
b = 8
b = 30
>>> b
30
```

## Closures
* A closure is a function with an extended scope that encompasses nonglobal variables referenced in the body of the function but not defined there. It does not matter whether the function is anonymous or not; what matters is that it can access nonglobal variables that are defined outside of its body.
  * Free Variable: a variable that is not bound in the local scope.
  * A closure is a function that retains the bindings of the free variables (`series` in the example) that exist when the function is defined, so that they can be used later when the function is invoked and the defining scope is no longer available.
  
* In following example, line #3 - #7 forms a closure, note this includes the declaration for `series`.
  * Python keeps the names of local and free variables in the `__code__` attribute that represents the compiled body of the function.
  * Each item in `avg.__closure__` corresponds to a name in `avg.__code__.co_freevars`. These items are `cells`, and they have an attribute called `cell_contents` where the actual value can be found.

```
def make_averager():                              #1
                                                  #2
    series = []  # series is a free variable      #3
    def averager(new_value):                      #4
        series.append(new_value)                  #5
        total = sum(series)                       #6
        return total/len(series)                  #7
                                                  #8
    return averager                               #9

>>> avg = make_averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0

>>> avg.__code__.co_varnames
('new_value', 'total')
>>> avg.__code__.co_freevars
('series',)
>>> avg.__code__.co_freevars
('series',)
>>> avg.__closure__
(<cell at 0x107a44f78: list object at 0x107a91a48>,)
>>> avg.__closure__[0].cell_contents
[10, 11, 12]
```

* Note that the only situation in which a function may need to deal with external variables that are nonglobal is when it is nested in another function.

### The nonlocal Declaration
* In the inner function, if you try to assign to or update a free variable, which is of immutable type (like numbers, strings, tuples, etc.), then you are implicitly creating a local variable. It is no longer a free variable, and therefore it is not saved in the closure.
* The *nonlocal declaration* lets you flag a variable as a free variable even when it is assigned a new value within the function. If a new value is assigned to a nonlocal variable, the binding stored in the closure is changed. 
```
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        nonlocal count, total   #without this declaration, following += will throw out UnboundLocalError error
        count += 1
        total += new_value
        return total / count

    return averager

```

## Decorators in the Standard Library
* Python has three built-in functions that are designed to decorate methods: property, classmethod, and staticmethod
* Another frequently seen decorator is functools.wraps, a helper for building well-behaved decorators.

### MEMOIZATION WITH FUNCTOOLS.LRU_CACHE
> Memoization: an optimization technique that works by saving the results of previous invocations of an expensive function, avoiding repeat computations on previously used arguments. 

* `functools.lru_cache` implements memoization. 
  * The letters LRU stand for Least Recently Used, meaning that the growth of the cache is limited by discarding the entries that have not been read for a while.

* `lru_cache` must be invoked as a regular function, `@functools.lru_cache()`, because it accepts configuration parameters.
  * `functools.lru_cache(maxsize=128, typed=False)`
  * The `maxsize` argument determines how many call results are stored. After the cache is full, older results are discarded to make room. For optimal performance, maxsize should be a power of 2. 
  * The `typed` argument, if set to True, stores results of different argument types separately, i.e., distinguishing between float and integer arguments that are normally considered equal, like 1 and 1.0. 
  * By the way, because `lru_cache` uses a dict to store the results, and the keys are made from the positional and keyword arguments used in the calls, all the arguments taken by the decorated function must be hashable.

* Applications
  * Make silly recursive algorithms viable
  * `lru_cache` really shines in applications that need to fetch information from the Web.

### GENERIC FUNCTIONS WITH SINGLE DISPATCH
* Suppose we want to write a program `htmlize` to generate HTML displays for different types of Python objects. 
* Because we don’t have method or function overloading in Python, we can’t create variations of `htmlize` with different signatures for each data type we want to handle differently. 
* A common solution in Python would be to turn `htmlize` into a *dispatch function*, with a chain of `if/elif/elif` calling specialized functions like `htmlize_str`, `htmlize_int`, etc. 
* This is not extensible by users of our module, and is unwieldy: over time, the `htmlize` dispatcher would become too big, and the coupling between it and the specialized functions would be very tight.

* `functools.singledispatch` decorator allows each module to contribute to the overall solution, and lets you easily provide a specialized function even for classes that you can’t edit. 
  * If you decorate a plain function with `@singledispatch`, it becomes a generic function: a group of functions to perform the same operation in different ways, depending on the type of the first argument (This is what is meant by the term single-dispatch. If more arguments were used to select the specific functions, we’d have multiple-dispatch.).
  * When possible, register the specialized functions to handle `ABC`s (abstract classes) such as `numbers.Integral` and `abc.MutableSequence` instead of concrete implementations like `int` and `list`. This allows your code to support a greater variety of compatible types. For example, a Python extension can provide alternatives to the `int` type with fixed bit lengths as subclasses of `numbers.Integral`.
  * Using ABCs for type checking allows your code to support existing or future classes that are either actual or virtual subclasses of those ABCs.

* A notable quality of the singledispatch mechanism is that you can register specialized functions anywhere in the system, in any module.
* `@singledispatch` is not designed to bring *Java-style* *method overloading* to Python. 
  * A single class with many overloaded variations of a method is better than a single function with a lengthy stretch of `if/elif/elif/elif` blocks. 
  * But both solutions are flawed because they concentrate too much responsibility in a single code unit—the class or the function. 
  * The advantage of `@singledispath` is supporting *modular extension*: each module can register a specialized function for each type it supports.
  
## Parameterized Decorators
* When parsing a decorator in source code, Python takes the decorated function and passes it as the first argument to the decorator function. 
* To make a decorator accept other arguments: make a **decorator factory** that takes those arguments and returns a decorator, which is then applied to the function to be decorated.
* Graham Dumpleton and Lennart Regebro argue that decorators are best coded as classes implementing `__call__`, and not as functions.
* Parameterized decorators almost aways involve at least two nested functions, maybe more if you want to use @functools.wraps to produce a decorator that provides better support for more advanced techniques. One such technique is stacked decorators.


# References
* Graham Dumpleton has [a series of in-depth blog posts](https://github.com/GrahamDumpleton/wrapt/blob/develop/blog/README.md) about techniques for implementing well-behaved decorators, starting with “How You Implemented Your Python Decorator is Wrong”. His deep expertise in this matter is also nicely packaged in the wrapt module he wrote to simplify the implementation of decorators and dynamic function wrappers, which support introspection and behave correctly when further decorated, when applied to methods and when used as descriptors.

* [PEP 443](https://www.python.org/dev/peps/pep-0443/) provides the rationale and a detailed description of the single-dispatch generic functions’ facility. An old (March 2005) blog post by Guido van Rossum, [“Five-Minute Multimethods in Python”](https://www.artima.com/weblogs/viewpost.jsp?thread=101605), walks through an implementation of generic functions (a.k.a. multimethods) using decorators. His code supports multiple-dispatch (i.e., dispatch based on more than one positional argument). Guido’s multimethods code is interesting, but it’s a didactic example. For a modern, production-ready implementation of multiple-dispatch generic functions, check out Reg by Martijn Faassen—author of the model-driven and REST-savvy Morepath web framework.

* [“Closures in Python”](http://effbot.org/zone/closure.htm) is a short blog post by Fredrik Lundh that explains the terminology of closures.

* PEP 3104 — Access to Names in Outer Scopes describes the introduction of the nonlocal declaration to allow rebinding of names that are neither local nor global. It also includes an excellent overview of how this issue is resolved in other dynamic languages (Perl, Ruby, JavaScript, etc.) and the pros and cons of the design options available to Python.

* On a more theoretical level, PEP 227 — Statically Nested Scopes documents the introduction of lexical scoping as an option in Python 2.1 and as a standard in Python 2.2, explaining the rationale and design choices for the implementation of closures in Python.


























  
