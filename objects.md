# Table of Contents
* [Object References](#object-references)
  * [Variables Are Labels, Not Boxes](#variables-are-labels-not-boxes)
* [Object Identity, Equality, and Aliasing](#object-identity-equality-and-aliasing)  
* [Shallow and Deep Copies](#shallow-and-deep-copies)
  * [Copies Are Shallow by Default](#copies-are-shallow-by-default)
  * [Deep and Shallow Copies of Arbitrary Objects](#deep-and-shallow-copies-of-arbitrary-objects)
* [Function Parameters as References](#function-parameters-as-references)
* [Garbage Collection](#garbage-collection)
* [User-Defined Object](#user-defined-object)
* [Sequence Hacking, Hashing, and Slicing](#sequence-hacking-hashing-and-slicing)
* [Interfaces: From Protocols to ABCs](#interfaces-from-protocols-to-abcs)

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

* The `==` operator compares the values of objects (the data they hold), while `is` compares their identities (references).
* However, if you are comparing a variable to a *singleton*, then it makes sense to use `is`. By far, the most common case is checking whether a variable is bound to `None`. 
```
x is None
```

* The `is` operator is faster than `==`, because it cannot be overloaded, so Python does not have to find and invoke special methods to evaluate it, and computing is as simple as comparing two integer IDs.
* In contrast, `a == b` is syntactic sugar for `a.__eq__(b)`. The `__eq__` method inherited from object compares object IDs, so it produces the same result as `is`. 
  * If you don’t override `__eq__`, the method inherited from object compares object IDs, so the fallback is that every instance of a user-defined class is considered different.
  * But most built-in types override `__eq__` with more meaningful implementations that actually take into account the values of the object attributes. *Equality* may involve a lot of processing—for example, when comparing large collections or deeply nested structures.

* Java Operator `==`
  * The == operator in Java never felt right for me. It is much more common for programmers to care about equality than identity, but for objects (not primitive types) the Java == compares references, and not object values. Even for something as basic as comparing strings, Java forces you to use the .equals method. 
  * Even then, there is another catch: if you write a.equals(b) and a is null, you get a null pointer exception. 
  * The Java designers felt the need to overload + for strings, so why not go ahead and overload == as well?
  * Python has operator overloading, == works sensibly with all objects in the standard library, including None, which is a proper object, unlike Java’s null.


### The Relative Immutability of Tuples
* A surprising trait of tuples: they are immutable but their values may change.
  * If the referenced items are mutable, they may change even if the tuple itself does not. 
  * In other words, the immutability of tuples really refers to the physical contents of the tuple data structure (i.e., the references it holds), and does not extend to the referenced objects.
  * What can never change in a tuple is the identity of the items it contains.
  * It's the reason about the [A += Assignment Puzzle](https://github.com/niuers/python/blob/master/python_sequences.md#a--assignment-puzzler)
  * It’s also the reason why some [tuples are unhashable](https://github.com/niuers/python/blob/master/dictionaries_and_sets.md#hashable-objects)

### Mutability
* When you are dealing with unchanging objects, it makes no difference whether variables hold the actual objects or references to shared objects. If `a == b` is true, and neither object can change, they might as well be the same. That’s why **string interning** is safe. 
* Object identity becomes important only when objects are mutable.
* In “pure” functional programming, all data is immutable: appending to a collection actually creates a new collection. 
* Python, however, is not a functional language, much less a pure one. 
  * Instances of user-defined classes are mutable by default in Python—as in most object-oriented languages. 
  * When creating your own objects, you have to be extra careful to make them immutable, if that is a requirement. 
  * Every attribute of the object must also be immutable, otherwise you end up with something like the tuple: immutable as far as object IDs go, but the value of a tuple may change if it holds a mutable object.
* Mutable objects are also the main reason why programming with threads is so hard to get right: threads mutating objects without proper synchronization produce corrupted data. Excessive synchronization, on the other hand, causes deadlocks.

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
## Other Parameter Passing Modes
* call by value (the function gets a copy of the argument) 
* call by reference (the function gets a pointer to the argument).

## Call by Sharing
* The only mode of parameter passing in Python is **call by sharing**. 
  * **Call by sharing** means that each formal parameter of the function gets a copy of each reference in the arguments. In other words, the parameters inside the function become aliases of the actual arguments.
  * That is the same mode used in most OO languages, including Ruby, SmallTalk, and Java (this applies to Java reference types; primitive types use call by value).
  
* The result of this scheme is that a function may change any mutable object passed as a parameter, but it cannot change the identity of those objects (i.e., it cannot altogether replace an object with another). 
* As we pass numbers, lists, and tuples to the function, the actual arguments passed are affected in different ways.
* In Python, the function gets a copy of the arguments, but the arguments are always references. So the value of the referenced objects may be changed, if they are mutable, but their identity cannot. Also, because the function gets a copy of the reference in an argument, rebinding it has no effect outside of the function. 


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
  * There is no mechanism in Python to directly destroy an object, and this omission is actually a great feature: if you could destroy an object at any time, what would happen to existing strong references pointing to it?

* The `del` statement deletes names, not objects. 
  * An object may be garbage collected as result of a `del` command, but only if the variable deleted holds the last reference to the object, or if the object becomes unreachable. (If two objects refer to each other, they may be destroyed if the garbage collector determines that they are otherwise unreachable because their only references are their mutual references.) 
  * Rebinding a variable may also cause the number of references to an object to reach zero, causing its destruction.

## CPython Implementation
* There is a `__del__` special method, but it does not cause the disposal of the instance, and should not be called by your code. `__del__` is invoked by the Python interpreter when the instance is about to be destroyed to give it a chance to release external resources. You will seldom need to implement `__del__` in your own code  
* In CPython, the primary algorithm for garbage collection is reference counting. Essentially, each object keeps count of how many references point to it. As soon as that refcount reaches zero, the object is immediately destroyed: CPython calls the `__del__` method on the object (if defined) and then frees the memory allocated to the object. 
* In CPython 2.0, a generational garbage collection algorithm was added to detect groups of objects involved in *reference cycles*—which may be unreachable even with outstanding references to them, when all the mutual references are contained within the group. 
* Other implementations of Python have more sophisticated garbage collectors that do not rely on reference counting, which means the `__del__` method may not be called immediately when there are no more references to the object. See [“PyPy, Garbage Collection, and a Deadlock”](https://emptysqua.re/blog/pypy-garbage-collection-and-a-deadlock/) by A. Jesse Jiryu Davis for discussion of improper and proper use of `__del__`.

## Other Python Implementations
* Jython or IronPython use the garbage collector of their host runtimes (the Java VM and the .NET CLR), which are more sophisticated but do not rely on reference counting and may take longer to destroy the object.
  * For example, the following line is safe in CPython, the reference count of the file object will be zero after the write method returns, and Python will immediately close the file before destroying the object representing it in memory.
  ```
  open('test.txt', 'wt', encoding='utf-8').write('1, 2, 3')
  ```
  * However, in Jython or IronPython, it may take longer to destroy the object and close the file.
  
* In all cases, including CPython, the best practice is to explicitly close the file, and the most reliable way of doing it is using the `with` statement, which guarantees that the file will be closed even if exceptions are raised while it is open.

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


## Further Reading
* Wesley Chun, author of the Core Python series of books, made a great presentation about many of the topics covered in this chapter during OSCON 2013. You can download the slides from the “Python 103: Memory Model & Best Practices” talk page. There is also a YouTube video of a longer presentation Wesley gave at EuroPython 2011, covering not only the theme of this chapter but also the use of special methods.

* Doug Hellmann wrote a long series of excellent blog posts titled Python Module of the Week, which became a book, The Python Standard Library by Example. His posts “copy – Duplicate Objects” and “weakref – Garbage-Collectable References to Objects” cover some of the topics we just discussed.

* More information on the CPython generational garbage collector can be found in the gc module documentation, which starts with the sentence “This module provides an interface to the optional garbage collector.” The “optional” qualifier here may be surprising, but the “Data Model” chapter also states:

* An implementation is allowed to postpone garbage collection or omit it altogether—it is a matter of implementation quality how garbage collection is implemented, as long as no objects are collected that are still reachable.

* Fredrik Lundh—creator of key libraries like ElementTree, Tkinter, and the PIL image library—has a short post about the Python garbage collector titled “How Does Python Manage Memory?” He emphasizes that the garbage collector is an implementation feature that behaves differently across Python interpreters. For example, Jython uses the Java garbage collector.

* The CPython 3.4 garbage collector improved handling of objects with a `__del__` method, as described in PEP 442 — Safe object finalization.

* Wikipedia has an article about string interning, mentioning the use of this technique in several languages, including Python.

* If you are into the subject of garbage collectors, you may want to read Thomas Perl’s paper “Python Garbage Collector Implementations: CPython, PyPy and GaS”, from which I learned the bit about the safety of the open().write() in CPython.

  


# User-Defined Object
* Thanks to the Python data model, your user-defined types can behave as naturally as the built-in types. And this can be accomplished without inheritance, in the spirit of duck typing: you just implement the methods needed for your objects to behave as expected.

## Object Representations
* An early realization of the need for distinct string representations for objects appeared in [Smalltalk](http://esug.org/data/HistoricalDocuments/TheSmalltalkReport/ST07/04wo.pdf).
  * For example, To a programmer, one of an object’s most important properties is its class.
* repr(): Return a string representing the object as the developer wants to see it.
  * Because of its role in debugging, calling repr() on an object should never raise an exception. If something goes wrong inside your implementation of `__repr__`, you must deal with the issue and do your best to produce some serviceable output that gives the user a chance of identifying the target object.

* str(): Return a string representing the object as the user wants to see it.

### classmethod Versus staticmethod
* The `classmethod` is used to define a method that operates on the class and not on instances. `classmethod` changes the way the method is called, so it receives the class itself as the first argument (usually named `cls`), instead of an instance. Its most common use is for alternative constructors.
  * No matter how you invoke it, class method receives the class as the first argument.
* The `staticmethod` decorator changes a method so that it receives no special first argument. In essence, a static method is just like a plain function that happens to live in a class body, instead of being defined at the module level.
  * The classmethod decorator is clearly useful, but I’ve never seen a compelling use case for staticmethod. If you want to define a function that does not interact with the class, just define it in the module.

### Formatted Displays
* The `format()` built-in function and the `str.format()` method delegate the actual formatting to each type by calling their `.__format__(format_spec)` method. The `format_spec` is a formatting specifier, which is either:
  * The second argument in `format(my_obj, format_spec)`, or
  * Whatever appears after the colon in a replacement field delimited with {} inside a format string used with `str.format()`

* A format string such as '{0.mass:5.3e}' actually uses two separate notations. The '0.mass' to the left of the colon is the field_name part of the replacement field syntax; the '5.3e' after the colon is the formatting specifier. The notation used in the formatting specifier is called the Format Specification Mini-Language.
  * The Format Specification Mini-Language is extensible because each class gets to interpret the format_spec argument as it likes.
* If a class has no `__format__`, the method inherited from object returns str(my_object).

## Hashable Objects
* To make an user-defined Object hashable, we must implement `__hash__` (`__eq__` is also required, and we already have it). We also need to make the instances immutable.
* `__hash__` is usually implemented using the recommended technique of xor-ing the hashes of the instance attributes.

## Private and “Protected” Attributes in Python
* Attributes are public in all Python instance and class by default.
* When (or if) we later need to impose more control with getters and setters, these can be implemented through properties without changing any of the code that already interacts with our objects through the names (e.g., x and y) that were initially simple public attributes.
* This approach is the opposite of that encouraged by the Java language: a Java programmer cannot start with simple public attributes and only later, if needed, implement properties, because they don’t exist in the language. Therefore, writing getters and setters is the norm in Java—even when those methods do nothing useful—because the API cannot evolve from simple public attributes to getters and setters without breaking all code that uses those attributes.
* Ward Cunningham, inventor of the wiki and an Extreme Programming pioneer, recommends asking “What’s the simplest thing that could possibly work?” The idea is to focus on the goal.[59] Implementing setters and getters up front is a distraction from the goal. In Python, we can simply use public attributes knowing we can change them to properties later, if the need arises.

* The Java private and protected modifiers normally provide protection against accidents only (i.e., safety). They can only guarantee security against malicious intent if the application is deployed with a security manager, and that seldom happens in practice, even in corporate settings.
  * That script uses introspection (“reflection” in Java parlance) to get the value of a private field.
  * Java reflection API can get a reference to the private field
  * My point is: in Java too, access control modifiers are mostly about safety and not security, at least in practice. 
  


### Name Mangling
* In Python, there is no way to create private variables like there is with the private modifier in Java. What we have in Python is a simple mechanism to prevent accidental overwriting of a “private” attribute in a subclass.
  * To prevent this, if you name an instance attribute in the form `__mood` (two leading underscores and zero or at most one trailing underscore), Python stores the name in the instance `__dict__` prefixed with a leading underscore and the class name, so in the `Dog` class, `__mood` becomes `_Dog__mood`, and in `Beagle` class it’s `_Beagle__mood`. This language feature goes by the lovely name of name mangling.
* Name mangling is about safety, not security: it’s designed to prevent accidental access and not intentional wrongdoing 

* Some prefer to avoid this syntax and use just one underscore prefix to “protect” attributes by convention (e.g., self._x). 
* Critics of the automatic double-underscore mangling suggest that concerns about accidental attribute clobbering should be addressed by naming conventions. This is the full quote from the prolific Ian Bicking, cited at the beginning of this chapter:
> Never, ever use two leading underscores. This is annoyingly private. If name clashes are a concern, use explicit name mangling instead (e.g., `_MyThing_blahblah`). This is essentially the same thing as double-underscore, only it’s transparent where double underscore obscures.

### Single Underscored Method
* The single underscore prefix has no special meaning to the Python interpreter when used in attribute names, but it’s a very strong convention among Python programmers that you should not access such attributes from outside the class.
* It’s easy to respect the privacy of an object that marks its attributes with a single `_`, just as it’s easy respect the convention that variables in ALL_CAPS should be treated as constants.
* In modules, a single _ in front of a top-level name does have an effect: if you write from mymod import * the names with a _ prefix are not imported from mymod. However, you can still write from mymod import `_privatefunc`.

## Saving Space with the __slots__ Class Attribute
* By default, Python stores instance attributes in a per-instance dict named `__dict__`. 
* Dictionaries have a significant memory overhead because of the underlying hash table used to provide fast access. 
* If you are dealing with millions of instances with few attributes, the `__slots__` class attribute can save a lot of memory, by letting the interpreter store the instance attributes in a `tuple` instead of a `dict`.
  * If you are handling millions of objects with numeric data, you should really be using NumPy arrays, which are not only memory-efficient but have highly optimized functions for numeric processing, many of which operate on the entire array at once.
* A `__slots__` attribute inherited from a superclass has no effect. Python only takes into account `__slots__` attributes defined in each class individually.
* I like to use a tuple for that, because it conveys the message that the `__slots__` definition cannot change.
* It may be possible, however, to “save memory and eat it too”: if you add the '__dict__' name to the `__slots__` list, your instances will keep attributes named in `__slots__` in the per-instance tuple, but will also support dynamically created attributes, which will be stored in the usual `__dict__`. Of course, having '__dict__' in `__slots__` may entirely defeat its purpose, depending on the number of static and dynamic attributes in each instance and how they are used. Careless optimization is even worse than premature optimization.
* if the class defines `__slots__`, and you need the instances to be targets of weak references, then you need to include '__weakref__' among the attributes named in `__slots__`.

### THE PROBLEMS WITH `__SLOTS__`
* To summarize, `__slots__` may provide significant memory savings if properly used, but there are a few caveats:
  * You must remember to redeclare `__slots__` in each subclass, because the inherited attribute is ignored by the interpreter.
  * Instances will only be able to have the attributes listed in `__slots__`, unless you include '__dict__' in `__slots__` (but doing so may negate the memory savings).
  * Instances cannot be targets of weak references unless you remember to include '__weakref__' in `__slots__`.
  * If your program is not handling millions of instances, it’s probably not worth the trouble of creating a somewhat unusual and tricky class whose instances may not accept dynamic attributes or may not support weak references. Like any optimization, `__slots__` should be used only if justified by a present need and when its benefit is proven by careful profiling.
* using `__slots__` just to prevent instance attribute creation is not recommended. `__slots__` should be used only to save memory, and only if that is a real issue.

## Overriding Class Attributes
* A distinctive feature of Python is how class attributes can be used as default values for instance attributes.
* But if you write to an instance attribute that does not exist, you create a new instance attribute—e.g., a typecode instance attribute—and the class attribute by the same name is untouched. 
  * However, from then on, whenever the code handling that instance reads `self.typecode`, the instance `typecode` will be retrieved, effectively **shadowing** the class attribute by the same name. 
  * This opens the possibility of customizing an individual instance with a different typecode.
* If you want to change a class attribute you must set it on the class directly, not through an instance. You could change the default typecode for all instances (that don’t have their own typecode) by doing this:
```
>>> Vector2d.typecode = 'f'
```
* However, there is an idiomatic Python way of achieving a more permanent effect, and being more explicit about the change. Because class attributes are public, they are inherited by subclasses, so it’s common practice to subclass just to customize a class data attribute.

## `__index__`
* `__index__` is used to coerce an object to an integer index in the specific context of sequence slicing, and was created to solve a need in NumPy.
* If you are curious about it, A.M. Kuchling’s What’s New in Python 2.5 has a short explanation, and PEP 357 — Allowing Any Object to be Used for Slicing details the need for __index__, from the perspective of an implementor of a C-extension, Travis Oliphant, the lead author of NumPy.


# Sequence Hacking, Hashing, and Slicing

## Protocols and Duck Typing
* You don’t need to inherit from any special class to create a fully functional sequence type in Python; you just need to implement the methods that fulfill the sequence protocol. But what kind of protocol are we talking about?
* In the context of object-oriented programming, a protocol is an informal interface, defined only in documentation and not in code. 
  * For example, the sequence protocol in Python entails just the `__len__` and `__getitem__` methods. Any class Spam that implements those methods with the standard signature and semantics can be used anywhere a sequence is expected. Whether Spam is a subclass of this or that is irrelevant; all that matters is that it provides the necessary methods.
  * We say it is a sequence because it behaves like one, and that is what matters.
  * protocols are defined as the informal interfaces that make polymorphism work in languages with dynamic typing like Python.
* This became known as **duck typing**, after Alex Martelli’s post quote.
  * Don’t check whether it is a duck: check whether it quacks-like-a duck, walks-like-a duck, etc, etc, depending on exactly what subset of duck-like behavior you need to play your language-games with. [comp.lang.python, Jul. 26, 2000, Alex Martelli](https://mail.python.org/pipermail/python-list/2000-July/046184.html)
  * The dynamic protocols are the hallmark of duck typing
  * In the Python documentation, you can often tell when a protocol is being discussed when you see language like “a file-like object.” This is a quick way of saying “something that behaves sufficiently like a file, by implementing the parts of the file interface that are relevant in the context.”
  * When implementing a class that emulates any built-in type, it is important that the emulation only be implemented to the degree that it makes sense for the object being modeled. For example, some sequences may work well with retrieval of individual elements, but extracting a slice may not make sense.
  * When we don’t need to code nonsense methods just to fulfill some over-designed interface contract and keep the compiler happy, it becomes easier to follow the **KISS principle**.
  * Duck typing is operating with objects regardless of their types, as long as they implement certain protocols.
  * Duck typing: ignoring an object’s actual type, focusing instead on ensuring that the object implements the method names, signatures, and semantics required for its intended use
  * In Python, this mostly boils down to avoiding the use of isinstance to check the object’s type (not to mention the even worse approach of checking, for example, whether type(foo) is bar—which is rightly anathema as it inhibits even the simplest forms of inheritance!).

* Established protocols naturally evolve in any language that uses dynamic typing, that is, when type-checking done at runtime because there is no static type information in method signatures and variables. Ruby is another important OO language that has dynamic typing and uses protocols.


### HOW SLICING WORKS
* Built-in `slice` method `indices` exposes the tricky logic that’s implemented in the built-in sequences to gracefully handle missing or negative indices and slices that are longer than the target sequence. This method produces “normalized” tuples of nonnegative start, stop, and stride integers adjusted to fit within the bounds of a sequence of the given length.
* Example considering a sequence of len == 5, e.g., 'ABCDE':
```
>>> slice(None, 10, 2).indices(5)  # 'ABCDE'[:10:2] is the same as 'ABCDE'[0:5:2]
(0, 5, 2)
>>> slice(-3, None, None).indices(5)  # 'ABCDE'[-3:] is the same as 'ABCDE'[2:5:1]
(2, 5, 1)
```
#### A SLICE-AWARE __GETITEM__
* This is not slice-aware
```
    def __getitem__(self, index):
        return self._components[index]
```

* This is slice-aware
```
    def __getitem__(self, index):
        cls = type(self)
        if isinstance(index, slice):
            return cls(self._components[index]) #invoke the class to build another Vector instance from a slice of the _components array
        elif isinstance(index, numbers.Integral):
            return self._components[index]   5
        else:
            msg = '{cls.__name__} indices must be integers'
            raise TypeError(msg.format(cls=cls))   6
```
## Dynamic Attribute Access
* The `__getattr__` method is invoked by the interpreter when attribute lookup fails. In simple terms, given the expression `my_obj.x`, Python checks if the `my_obj` instance has an attribute named `x`; if not, the search goes to the class (`my_obj.__class__`), and then up the inheritance graph. If the `x` attribute is not found, then the `__getattr__` method defined in the class of `my_obj` is called with self and the name of the attribute as a string (e.g., 'x').
* However, Python only calls that method as a fall back, when the object does not have the named attribute. However, after we assign `my_obj.x = 10`, the `my_obj` object now has an `x` attribute, so `__getattr__` will no longer be called to retrieve `my_obj.x`: the interpreter will just return the value 10 that is bound to `my_obj.x`.
* very often when you implement `__getattr__` you need to code `__setattr__` as well, to avoid inconsistent behavior in your objects.

## Hashing and a Faster ==
* Reduce
  * The first argument to reduce() is a two-argument function, and the second argument is an iterable. Let’s say we have a two-argument function fn and a list lst. When you call reduce(fn, lst), fn will be applied to the first pair of elements—fn(lst[0], lst[1])—producing a first result, r1. Then fn is applied to r1 and the next element—fn(r1, lst[2])—producing a second result, r2. Now fn(r2, lst[3]) is called to produce r3 … and so on until the last element, when a single result, rN, is returned.
  * When using reduce, it’s good practice to provide the third argument, reduce(function, iterable, initializer), to prevent this exception: TypeError: reduce() of empty sequence with no initial value (excellent message: explains the problem and how to fix it). The initializer is the value returned if the sequence is empty and is used as the first argument in the reducing loop, so it should be the identity value of the operation. As examples, for +, |, ^ the initializer should be 0, but for *, & it should be 1.
  * The powerful reduce higher-order function is also known as fold, accumulate, aggregate, compress, and inject. For more information, see Wikipedia’s [“Fold (higher-order function)”](https://en.wikipedia.org/wiki/Fold_(higher-order_function)) article, which presents applications of that higher-order function with emphasis on functional programming with recursive data structures. The article also includes a table listing fold-like functions in dozens of programming languages.
  * In the same conversation, Alex Martelli suggests the reduce built-in in Python 2 was more trouble than it was worth, because it encouraged coding idioms that were hard to explain. He was most convincing: the function was demoted to the functools module in Python 3.

* The `__hash__` method is a perfect example of a map-reduce computation.
* In Python 3, `map` is lazy: it creates a generator that yields the results on demand, thus saving memory—just like the generator expression. `map` would be less efficient in Python 2, where the map function builds a new list with the results.

# Interfaces: From Protocols to ABCs
## Interfaces and Protocols in Python Culture
* How do interfaces work in a dynamic-typed language? 
  * First, the basics: even without an *interface* keyword in the language, and regardless of ABCs, every class has an interface: the set public attributes (methods or data attributes) implemented or inherited by the class. This includes special methods, like `__getitem__` or `__add__`.
  * By definition, protected and private attributes are not part of an interface, even if “protected” is merely a naming convention (the single leading underscore) and private attributes are easily accessed. It is bad form to violate these conventions.
  * On the other hand, it’s not a sin to have public data attributes as part of the interface of an object, because—if necessary—a data attribute can always be turned into a property implementing getter/setter logic without breaking client code that uses the plain `obj.attr` syntax.
  
* A useful complementary definition of interface is: the subset of an object’s public methods that enable it to play a specific role in the system. 
  * That’s what is implied when the Python documentation mentions “a file-like object” or “an iterable,” without specifying a class. 
  * An interface seen as a set of methods to fulfill a role is what Smalltalkers called a procotol, and the term spread to other dynamic language communities. 
  * Protocols are independent of inheritance. A class may implement several protocols, enabling its instances to fulfill several roles.
  * Protocols are interfaces, but because they are informal—defined only by documentation and conventions—protocols cannot be enforced like formal interfaces can (However ABCs enforce interface conformance). A protocol may be partially implemented in a particular class, and that’s OK.
  * “X-like object,” “X protocol,” and “X interface” are synonyms in the minds of Pythonistas.
  * One of the most fundamental interfaces in Python is the sequence protocol.
### Python Digs Sequences
* The philosophy of the Python data model is to cooperate with essential protocols as much as possible. When it comes to sequences, Python tries hard to work with even the simplest implementations.
* In summary, given the importance of the sequence protocol, in the absence __iter__ and __contains__ Python still manages to make iteration and the in operator work by invoking __getitem__.
* Iteration in Python represents an extreme form of duck typing: the interpreter tries two different methods to iterate over objects.
  * When there is no method `__iter__`, as a fallback, when Python sees a `__getitem__` method, it tries to iterate over the object by calling that method with integer indexes starting with 0. 
  * Python is smart enough to make the `in` operator work even if there's no `__contains__` method: it does a full scan to check if an item is present.

### Monkey-Patching to Implement a Protocol at Runtime
* When you follow established protocols, you improve your chances of leveraging existing standard library and third-party code, thanks to duck typing.
* Monkey patching: changing a class or module at runtime, without touching the source code. Monkey patching is powerful, but the code that does the actual patching is very tightly coupled with the program to be patched, often handling private and undocumented parts.
```
>>> def set_card(deck, position, card):
...     deck._cards[position] = card
...
>>> FrenchDeck.__setitem__ = set_card
>>> shuffle(deck)
>>> deck[:5]
[Card(rank='3', suit='hearts'), Card(rank='4', suit='diamonds'), Card(rank='4',
suit='clubs'), Card(rank='7', suit='hearts'), Card(rank='9', suit='spades')]
```
* protocols are dynamic: random.shuffle doesn’t care what type of argument it gets, it only needs the object to implement part of the mutable sequence protocol (i.e. `__setitem__` method). It doesn’t even matter if the object was “born” with the necessary methods or if they were somehow acquired later.

## ABC
* ABCs, like descriptors and metaclasses, are tools for building frameworks. Therefore, only a very small minority of Python developers can create ABCs without imposing unreasonable limitations and needless work on fellow programmers.

### Alex Martelli’s Waterfowl
* I’m recommending supplementing (not entirely replacing—in certain contexts it shall still serve) good old duck typing with… goose typing!
* What goose typing means is: isinstance(obj, cls) is now just fine… as long as cls is an abstract base class—in other words, cls’s metaclass is abc.ABCMeta.
  * You can find many useful existing abstract classes in collections.abc (and additional ones in the numbers module of The Python Standard Library).

* Python’s ABCs add one major practical advantage: the `register` class method, which lets end-user code “declare” that a certain class becomes a “virtual” subclass of an ABC (for this purpose the registered class must meet the ABC’s method name and signature requirements, and more importantly the underlying semantic contract—but it need not have been developed with any awareness of the ABC, and in particular need not inherit from it!). 
  * This goes a long way toward breaking the rigidity and strong coupling that make inheritance something to use with much more caution than typically practiced by most OOP programmers…
  * Sometimes you don’t even need to register a class for an ABC to recognize it as a subclass!
* That’s the case for the ABCs whose essence boils down to a few special methods.
  * whenever you’re implementing a class embodying any of the concepts represented in the ABCs in numbers, collections.abc, or other framework you may be using, be sure (if needed) to subclass it from, or register it into, the corresponding ABC. At the start of your programs using some library or framework defining classes which have omitted to do that, perform the registrations yourself; then, when you must check for (most typically) an argument being, e.g, “a sequence,” check whether:
  ```
  isinstance(the_arg, collections.abc.Sequence)
  ```
 * And, don’t define custom ABCs (or metaclasses) in production code… if you feel the urge to do so, I’d bet it’s likely to be a case of “all problems look like a nail”-syndrome for somebody who just got a shiny new hammer—you (and future maintainers of your code) will be much happier sticking with straightforward and simple code, eschewing such depths. Valē!
* Alex makes the point that inheriting from an ABC is more than implementing the required methods: it’s also a clear declaration of intent by the developer. That intent can also be made explicit through registering a virtual subclass.

* In addition, the use of `isinstance` and `issubclass` becomes more acceptable to test against ABCs. In the past, these functions worked against duck typing, but with ABCs they become more flexible. After all, if a component does not implement an ABC by subclassing, it can always be registered after the fact so it passes those explicit type checks.

* However, even with ABCs, you should beware that excessive use of isinstance checks may be a **code smell**—a symptom of bad OO design. It’s usually not OK to have a chain of if/elif/elif with insinstance checks performing different actions depending on the type of an object: you should be using polymorphism for that—i.e., designing your classes so that the interpreter dispatches calls to the proper methods, instead of you hardcoding the dispatch logic in if/elif/elif blocks.
* There is a common, practical exception to the preceding recommendation: some Python APIs accept a single str or a sequence of str items; if it’s just a single str, you want to wrap it in a list, to ease processing. Because str is a sequence type, the simplest way to distinguish it from any other immutable sequence is to do an explicit isinstance(x, str) check.

* On the other hand, it’s usually OK to perform an insinstance check against an ABC if you must enforce an API contract: “Dude, you have to implement this if you want to call me,” as technical reviewer Lennart Regebro put it. That’s particularly useful in systems that have a plug-in architecture. Outside of frameworks, duck typing is often simpler and more flexible than type checks.

* For example, in several classes in this book, when I needed to take a sequence of items and process them as a list, instead of requiring a list argument by type checking, I simply took the argument and immediately built a list from it: that way I can accept any iterable, and if the argument is not iterable, the call will fail soon enough with a very clear message. One example of this code pattern is in the __init__ method in Example 11-13, later in this chapter. Of course, this approach wouldn’t work if the sequence argument shouldn’t be copied, either because it’s too large or because my code needs to change it in place. Then an insinstance(x, abc.MutableSequence) would be better. If any iterable is acceptable, then calling iter(x) to obtain an iterator would be the way to go

* Finally, in his essay, Alex reinforces more than once the need for restraint in the creation of ABCs. An ABC epidemic would be disastrous, imposing excessive ceremony in a language that became popular because it’s practical and pragmatic.

* ABCs are meant to encapsulate very general concepts, abstractions, introduced by a framework—things like “a sequence” and “an exact number.” [Readers] most likely don’t need to write any new ABCs, just use existing ones correctly, to get 99.9% of the benefits without serious risk of misdesign.

* EAFP = it’s easier to ask forgiveness than permission

### Subclassing an ABC
### ABCs in the Standard Library
#### The `numbers` Package
* The numbers package defines the so-called “numerical tower” (i.e., this linear hierarchy of ABCs), where Number is the topmost superclass, Complex is its immediate subclass, and so on, down to Integral:
  * Number
  * Complex
  * Real
  * Rational
  * Integral

### Defining and Using an ABC
* The best way to declare an ABC is to subclass abc.ABC or any other ABC.
* Concrete methods in an ABC must rely only on the interface defined by the ABC (i.e., other concrete or abstract methods or properties of the ABC).
* An abstract method can actually have an implementation. Even if it does, subclasses will still be forced to override it, but they will be able to invoke the abstract method with super(), adding functionality to it instead of implementing from scratch.
* An ABC is a special kind of class; for example, “regular” classes don’t check subclasses, so this is a special behavior of ABCs.(???)
* The order of stacked function decorators usually matters, and in the case of @abstractmethod, the documentation is explicit: 
  * When abstractmethod() is applied in combination with other method descriptors, it should be applied as the innermost decorator
  * In other words, no other decorator may appear between @abstractmethod and the def statement.

* An idiom worth mentioning: 
  * In `__init__`, `self._balls` stores `list(iterable)` and not just a reference to iterable (i.e., we did not merely assign iterable to `self._balls`). 
    * This makes our `LotteryBlower` flexible because the `iterable` argument may be any iterable type. 
    * At the same time, we make sure to store its items in a list so we can pop items. And even if we always get lists as the iterable argument, list(iterable) produces a copy of the argument, which is a good practice considering we will be removing items from it and the client may not be expecting the list of items she provided to be changed.
  ```
  import random
  from tombola import Tombola
  class LotteryBlower(Tombola):
    def __init__(self, iterable):
        self._balls = list(iterable)
  ```

* Declaring virtual subclasses with the register method
  * An essential characteristic of goose typing—and the reason why it deserves a waterfowl name—is the ability to register a class as a virtual subclass of an ABC, even if it does not inherit from it. When doing so, we promise that the class faithfully implements the interface defined in the ABC—and Python will believe us without checking. If we lie, we’ll be caught by the usual runtime exceptions.
  * This is done by calling a register method on the ABC. The registered class then becomes a virtual subclass of the ABC, and will be recognized as such by functions like issubclass and isinstance, but it will not inherit any methods or attributes from the ABC.
  * Virtual subclasses do not inherit from their registered ABCs, and are not checked for conformance to the ABC interface at any time, not even when they are instantiated (Compare with subclasses of ABCs that do get checked). It’s up to the subclass to actually implement all the methods needed to avoid runtime errors.

* Inheritance is guided by a special class attribute named `__mro__`—the Method Resolution Order. It basically lists the class and its superclasses in the order Python uses to search for methods. If you inspect the `__mro__`, you’ll see that it lists only the “real” superclasses.
* Even if `register` can now be used as a decorator, it’s more widely deployed as a function to register classes defined elsewhere. 

### Geese Can Behave as Ducks







