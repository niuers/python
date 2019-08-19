# Table of Contents
* [First-Class Objects](#first-class-objects)
  * [Hallmarks of Functional Programming](#hallmarks-of-functional-programming)
  * [Treat Function like an Object](#treat-function-like-an-object)
  * [High Order Functions](#high-order-functions)
* [](#)
* [](#)
* [](#)
* [](#)


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

## Hallmarks of Functional Programming
* First-Class Functions
* Higher Order Functions
> A function that takes a function as argument or returns a function as the result is a higher-order function.

* Immutable Data
* Pure Functions
* Recursion
* Manipulation of Lists
* Lazy Evaluation

## Treat Function like an Object
* Function object itself is an instance of the `function` class.

## High Order Functions
* `map` and `filter`: A `listcomp` or a `genexp` does the job of `map` and `filter` combined, but is more readable.
  * In Python 3, `map` and `filter` return `generators`—a form of `iterator`—so their direct substitute is now a `generator expression` (in Python 2, these functions returned lists, therefore their closest alternative is a `listcomp`).
* `reduce`: It was demoted from a built-in in Python 2 to the `functools` module in Python 3.
  * Its most common use case, `summation`, is better served by the `sum` built-in. This is a big win in terms of readability and performance.
  * Other reducing built-ins are `all` and `any`.
    * `all([])` returns True
    * `any([])` returns False
* `apply`: removed in Python 3

## Anonymous Functions




  
