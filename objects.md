# Table of Contents
* [Object References](#object-references)
  * [Variables Are Labels, Not Boxes](#variables-are-labels-not-boxes)
* [Object Identity, Equality, and Aliasing](#object-identity-equality-and-aliasing)  
* [Shallow and Deep Copies](#shallow-and-deep-copies)
  * [Copies Are Shallow by Default](#copies-are-shallow-by-default)
  * [Deep and Shallow Copies of Arbitrary Objects](#deep-and-shallow-copies-of-arbitrary-objects)
* [Function Parameters as References](#function-parameters-as-references)
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

## Object Identity

* Every object has an *identity*, a *type* and a *value*.
  * Only the value of an object changes over time. Actually the type of an object may be changed by merely assigning a different class to its `__class__` attribute, but that is pure evil.
  
* Because variables are mere labels, nothing prevents an object from having several labels assigned to it. When that happens, you have **aliasing**.
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

### The Relative Immutability of Tuples
* A surprising trait of tuples is revealed: they are immutable but their values may change.
  * If the referenced items are mutable, they may change even if the tuple itself does not. 
  * In other words, the immutability of tuples really refers to the physical contents of the tuple data structure (i.e., the references it holds), and does not extend to the referenced objects.
  * What can never change in a tuple is the identity of the items it contains.
  * It's the reason about the [A += Assignment Puzzle](https://github.com/niuers/python/blob/master/python_sequences.md#a--assignment-puzzler)
  * It’s also the reason why some [tuples are unhashable](https://github.com/niuers/python/blob/master/dictionaries_and_sets.md#hashable-objects)
  

# Shallow and Deep Copies
* The distinction between equality and identity has further implications when you need to copy an object. 
* A *copy* is an equal object with a different ID. But if an object contains other objects, should the copy also duplicate the inner objects, or is it OK to share them? There’s no single answer. 

## Copies Are Shallow by Default
* The easiest way to copy a list (or most built-in mutable collections) is to use the built-in constructor for the type itself.
```
l2 = list(l1)

#For lists and other mutable sequences, the shortcut below also makes a copy

l2 = l1[:] 
```

* However, using the constructor or `[:]` produces a shallow copy (i.e., the outermost container is duplicated, but the copy is filled with references to the same items held by the original container). This saves memory and causes no problems if all the items are immutable. But if there are mutable items, this may lead to unpleasant surprises.

## Deep and Shallow Copies of Arbitrary Objects
* The `copy` module provides the `deepcopy` and `copy` functions that return deep and shallow copies of arbitrary objects.
* Note that making deep copies is not a simple matter in the general case. Objects may have cyclic references that would cause a naïve algorithm to enter an infinite loop. The deepcopy function remembers the objects already copied to handle cyclic references gracefully.
* Also, a deep copy may be too deep in some cases. For example, objects may refer to external resources or singletons that should not be copied.

# Function Parameters as References
* The only mode of parameter passing in Python is **call by sharing**. 
  * **Call by sharing** means that each formal parameter of the function gets a copy of each reference in the arguments. In other words, the parameters inside the function become aliases of the actual arguments.
  * That is the same mode used in most OO languages, including Ruby, SmallTalk, and Java (this applies to Java reference types; primitive types use call by value).
  
* The result of this scheme is that a function may change any mutable object passed as a parameter, but it cannot change the identity of those objects (i.e., it cannot altogether replace an object with another). 
* As we pass numbers, lists, and tuples to the function, the actual arguments passed are affected in different ways.

### Mutable types as Parameter Defaults: Bad Idea
* The problem is that each default value is evaluated when the function is defined—i.e., usually when the module is loaded—and the default values become attributes (`__defaults__` attribute) of the function object. So if a default value is a mutable object, and you change it, the change will affect every future call of the function.
* The issue with mutable defaults explains why `None` is often used as the default value for parameters that may receive mutable values.
* Example: The problem here is that `Bus` instances that don’t get an initial passenger list end up sharing the same passenger list among themselves.

```
class HauntedBus:
    """A bus model haunted by ghost passengers"""

    #When the passengers argument is not passed, this parameter is bound to the default list object, which is initially empty.
    def __init__(self, passengers=[]): 
        # This assignment makes self.passengers an alias for passengers, which is itself an alias for the default list, when no passengers argument is given.
        self.passengers = passengers   

    def pick(self, name):
        # When the methods .remove() and .append() are used with self.passengers we are actually mutating the default list, which is an attribute of the function object (i.e. attribute of the function `__init__.__defaults__`).
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)
```
### Defensive Programming with Mutable Parameters
* When you are coding a function that receives a mutable parameter, you should carefully consider whether the caller expects the argument passed to be changed.
* Unless a method is explicitly intended to mutate an object received as argument, you should think twice before aliasing the argument object by simply assigning it to an instance variable in your class. If in doubt, make a copy. Your clients will often be happier.

* If two variables refer to immutable objects that have equal values (`a == b` is True), in practice it rarely matters if they refer to copies or are aliases referring to the same object because the value of an immutable object does not change, with one exception. 
  * The exception is immutable collections such as `tuples` and `frozensets`: if an immutable collection holds references to mutable items, then its value may actually change when the value of a mutable item changes. In practice, this scenario is not so common. What never changes in an immutable collection are the identities of the objects within.

* The fact that variables hold references has many practical consequences in Python programming:
  * Simple assignment does not create copies.
  * Augmented assignment with += or *= creates new objects if the lefthand variable is bound to an immutable object, but may modify a mutable object in place.
  * Assigning a new value to an existing variable does not change the object previously bound to it. 
  > This is called a rebinding: the variable is now bound to a different object. If that variable was the last reference to the previous object, that object will be garbage collected.
  * Function parameters are passed as aliases, which means the function may change any mutable object received as an argument. There is no way to prevent this, except making local copies or using immutable objects (e.g., passing a tuple instead of a list).
  * Using mutable objects as default values for function parameters is dangerous because if the parameters are changed in place, then the default is changed, affecting every future call that relies on the default.


# Garbage Collection
* Objects in Python are never explicitly destroyed; however, when they become unreachable they may be garbage-collected.
* The `del` statement deletes names, not objects. 
  * An object may be garbage collected as result of a `del` command, but only if the variable deleted holds the last reference to the object, or if the object becomes unreachable. (If two objects refer to each other, they may be destroyed if the garbage collector determines that they are otherwise unreachable because their only references are their mutual references.) 
  * Rebinding a variable may also cause the number of references to an object to reach zero, causing its destruction.

## CPython Implementation
* There is a `__del__` special method, but it does not cause the disposal of the instance, and should not be called by your code. `__del__` is invoked by the Python interpreter when the instance is about to be destroyed to give it a chance to release external resources. You will seldom need to implement `__del__` in your own code  
* In CPython, the primary algorithm for garbage collection is reference counting. Essentially, each object keeps count of how many references point to it. As soon as that refcount reaches zero, the object is immediately destroyed: CPython calls the `__del__` method on the object (if defined) and then frees the memory allocated to the object. 
* In CPython 2.0, a generational garbage collection algorithm was added to detect groups of objects involved in *reference cycles*—which may be unreachable even with outstanding references to them, when all the mutual references are contained within the group. 
* Other implementations of Python have more sophisticated garbage collectors that do not rely on reference counting, which means the `__del__` method may not be called immediately when there are no more references to the object. See [“PyPy, Garbage Collection, and a Deadlock”](https://emptysqua.re/blog/pypy-garbage-collection-and-a-deadlock/) by A. Jesse Jiryu Davis for discussion of improper and proper use of `__del__`.

## Weak References
* The presence of references is what keeps an object alive in memory. When the reference count of an object reaches zero, the garbage collector disposes of it. 
* But sometimes it is useful to have a reference to an object that does not keep it around longer than necessary. A common use case is a *cache*.
> Weak references to an object do not increase its reference count. The object that is the target of a reference is called *the referent*. Therefore, we say that a weak reference does not prevent the referent from being garbage collected.

* Weak references are useful in caching applications because you don’t want the cached objects to be kept alive just because they are referenced by the cache.

### Unexpected References to Objects
* The Python console automatically binds the `_` variable to the result of expressions that are not None. This highlights a practical matter: when trying to micro-manage memory we are often surprised by hidden, implicit assignments that create new references to our objects. The _ console variable is one example. 
* Traceback objects are another common source of unexpected references.

## The weakref Module
* The `weakref` module documentation makes the point that the `weakref.ref` class is actually a low-level interface intended for advanced uses, and that most programs are better served by the use of the weakref collections and `finalize`. 
* In other words, consider using `WeakKeyDictionary`, `WeakValueDictionary`, `WeakSet`, and finalize (which use weak references internally) instead of creating and handling your own weakref.ref instances by hand.

### WeakKeyDictionary
* The class WeakValueDictionary implements a mutable mapping where the values are weak references to objects. When a referred object is garbage collected elsewhere in the program, the corresponding key is automatically removed from WeakValueDictionary. This is commonly used for caching.

* A temporary variable may cause an object to last longer than expected by holding a reference to it. This is usually not a problem with local variables: they are destroyed when the function returns. 
  * But in Example below, the for loop variable cheese is a global variable and will never go away unless explicitly deleted.
```
>>> import weakref
>>> stock = weakref.WeakValueDictionary()  
>>> catalog = [Cheese('Red Leicester'), Cheese('Tilsit'),
...                 Cheese('Brie'), Cheese('Parmesan')]
...
# The for-loop variable cheese is a global variable here and will never go away unless explicitly deleted.
>>> for cheese in catalog:
...     stock[cheese.kind] = cheese  
...
>>> sorted(stock.keys())
['Brie', 'Parmesan', 'Red Leicester', 'Tilsit']  
>>> del catalog
>>> sorted(stock.keys())
['Parmesan']  # Why this one is left here? because 'cheese' is a global variable, which is a reference to 'Parmesan'
>>> del cheese
>>> sorted(stock.keys())
[]

```

### LIMITATIONS OF WEAK REFERENCES
* Not every Python object may be the target, or referent, of a weak reference. Basic `list` and `dict` instances may not be referents, but a plain subclass of either can solve this problem easily:
* A set instance can be a referent. 
* User-defined types also pose no problem.
* But `int` and `tuple` instances cannot be targets of weak references, even if subclasses of those types are created.
* Most of these limitations are implementation details of CPython that may not apply to other Python iterpreters. They are the result of internal optimizations.

### Tricks Python Plays with Immutables
* For a tuple t, `t[:]` does not make a copy, but returns a reference to the same object. You also get a reference to the same tuple if you write `tuple(t)`.
```
>>> t1 = (1, 2, 3)
>>> t2 = tuple(t1)
>>> t2 is t1
True
>>> t3 = t1[:]
>>> t3 is t1
True
```

* The same behavior can be observed with instances of `str`, `bytes`, and `frozenset`. Note that a `frozenset` is not a sequence, so `fs[:]` does not work if fs is a `frozenset`. 
  * But `fs.copy()` has the same effect: it cheats and returns a reference to the same object, and not a copy at all.
```
>>> t1 = (1, 2, 3)
>>> t3 = (1, 2, 3)  # 1
>>> t3 is t1  # 2: t1 and t3 are equal, but not the same object.
False
>>> s1 = 'ABC'
>>> s2 = 'ABC'  # 3
>>> s2 is s1 # 4: Surprise: a and b refer to the same str!
True
```

* The sharing of string literals is an optimization technique called **interning**. 
  * CPython uses the same technique with small integers to avoid unnecessary duplication of “popular” numbers like 0, –1, and 42. 
  * Note that CPython does not intern all strings or integers, and the criteria it uses to do so is an undocumented implementation detail.
* Never depend on str or int interning! Always use == and not is to compare them for equality. Interning is a feature for internal use of the Python interpreter.



  















