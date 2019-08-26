# Table of Contents
* [Object References](#object-references)
  * [Variables Are Labels, Not Boxes](#variables-are-labels-not-boxes)
* [Shallow and Deep Copies](#shallow-and-deep-copies)
* [References and Functions](#references-and-functions)
* [Garbage Collection](#garbage-collection)

# Object References
## Variables Are Labels, Not Boxes
* The distinction between objects and their names. A name is not the object; a name is a separate thing.
  * The usual “variables as boxes” metaphor actually hinders the understanding of reference variables in OO languages. 
  * Python variables are like reference variables in Java, so it’s better to think of them as labels attached to objects.
* With reference variables, it makes much more sense to say that the variable is assigned to an object, and not the other way around. After all, the object is created before the assignment.
* To understand an assignment in Python, always read the right-hand side first: that’s where the object is created or retrieved. After that, the variable on the left is bound to the object, like a label stuck to it. Just forget about the boxes.
* Single-type sequences like `str`, `bytes`, and `array.array` are flat: they don’t contain references but physically hold their data—characters, bytes, and numbers—in contiguous memory.
* `Tuples`, like most Python collections—`lists`, `dicts`, `sets`, etc.—hold references to objects.

# Object Identity, Equality, and Aliasing

* Every object has an *identity*, a *type* and a *value*. 

## Object Identity

* An object’s *identity* is guaranteed to be a **unique numeric label**, and it will **never change** during the life of the object.
  * You may think of it as the object’s address in memory.
  * The `is` operator compares the identity of two objects; the `id()` function returns an integer representing its identity.
  * The real meaning of an object’s ID is implementation-dependent. 
    * In CPython, `id()` returns the memory address of the object
    * But it may be something else in another Python interpreter. 
  * In practice, we rarely use the `id()` function while programming. Identity checks are most often done with the `is` operator, and not by comparing IDs.
* Two variables can be assigned to different objects (`is` will return `False`), but their values are the same (`==` returns `True`).  

### Choosing Between `==` and `is`

* The `==` operator compares the values of objects (the data they hold), while `is` compares their identities.
* However, if you are comparing a variable to a *singleton*, then it makes sense to use `is`. By far, the most common case is checking whether a variable is bound to `None`. 
```
x is None
```

* The `is` operator is faster than `==`, because it cannot be overloaded, so Python does not have to find and invoke special methods to evaluate it, and computing is as simple as comparing two integer IDs.
* In contrast, `a == b` is syntactic sugar for `a.__eq__(b)`. The `__eq__` method inherited from object compares object IDs, so it produces the same result as `is`. 
  * But most built-in types override `__eq__` with more meaningful implementations that actually take into account the values of the object attributes. *Equality* may involve a lot of processing—for example, when comparing large collections or deeply nested structures.

### THE RELATIVE IMMUTABILITY OF TUPLES
* A surprising trait of tuples is revealed: they are immutable but their values may change.
  * If the referenced items are mutable, they may change even if the tuple itself does not. 
  * In other words, the immutability of tuples really refers to the physical contents of the tuple data structure (i.e., the references it holds), and does not extend to the referenced objects.
  * What can never change in a tuple is the identity of the items it contains.
  * It’s also the reason why some tuples are unhashable



* Because variables are mere labels, nothing prevents an object from having several labels assigned to it. When that happens, you have aliasing.




# Shallow and Deep Copies

# References and Functions

* References and function parameters are our next theme: the problem with mutable parameter defaults and the safe handling of mutable arguments passed by clients of our functions.

# Garbage Collection
* The del command, and how to use weak references to “remember” objects without keeping them alive.





