# Table of Contents

* [](#)
* [](#)
* [](#)

# Coroutine
> A coroutine is a procedure that collaborates with the caller, yielding and receiving values from the caller.

* A coroutine is syntactically like a generator: just a function with the yield keyword in its body. 
* However, in a coroutine, yield usually appears on the right side of an expression (e.g., datum = yield), and it may or may not produce a value—if there is no expression after the yield keyword, the generator yields None. 
* The coroutine may receive data from the caller, which uses .send(datum) instead of next(…) to feed the coroutine. Usually, the caller pushes values into the coroutine.
* It is even possible that no data goes in or out through the yield keyword. 
* Regardless of the flow of data, yield is a **control flow device** that can be used to implement cooperative multitasking: each coroutine yields control to a central scheduler so that other coroutines can be activated.

* a broad, informal definition of a coroutine: a generator function driven by a client sending it data through .send(…) calls or yield from.
  * asyncio coroutines are (usually) decorated with an @asyncio.coroutine decorator, and they are always driven by yield from, not by calling .send(…) directly on them. Of course, asyncio coroutines are driven by next(…) and .send(…) under the covers, but in user code we only use yield from to make them run.



* We saw how the statement return the_result in a generator now raises StopIteration(the_result), allowing the caller to retrieve the_result from the value attribute of the exception. This is a rather cumbersome way to retrieve coroutine results, but it’s handled automatically by the yield from syntax introduced in PEP 380.


## Applications
* Coroutines are a natural way of expressing many algorithms, such as simulations, games, asynchronous I/O, and other forms of event-driven programming or co-operative multitasking. Simulation is a classic application of coroutines in the computer science literature.
* Coroutines are the fundamental building block of the asyncio package. A simulation shows how to implement concurrent activities using coroutines instead of threads.

* A discrete event simulation (DES) is a type of simulation where a system is modeled as a sequence of events. In a DES, the simulation “clock” does not advance by fixed increments, but advances directly to the simulated time of the next modeled event.
  * Intuitively, turn-based games are examples of discrete event simulations
  * In the field of simulation, the term process refers to the activities of an entity in the model, and not to an OS process. A simulation process may be implemented as an OS process, but usually a thread or a coroutine is used for that purpose.
  
* Both types of simulations can be written with multiple threads or a single thread using event-oriented programming techniques such as callbacks or coroutines driven by an event loop. It’s arguably more natural to implement a continuous simulation using threads to account for actions happening in parallel in real time. On the other hand, coroutines offer exactly the right abstraction for writing a DES. 
* The verb “drive” is commonly used to describe the operation of a coroutine: the client code drives the coroutine by sending it values.
* Priority queues are a fundamental building block of discrete event simulations: events are created in any order, placed in the queue, and later retrieved in order according to the scheduled time of each one.

* The running average example demonstrated a common use for a coroutine: as an accumulator processing items sent to it. We saw how a decorator can be applied to prime a coroutine, making it more convenient to use in some cases. But keep in mind that priming decorators are not compatible with some uses of coroutines. In particular, yield from subgenerator() assumes the subgenerator is not primed, and primes it automatically.

* In event-oriented programming with coroutines, each concurrent activity is carried out by a coroutine that repeatedly yields control back to the main loop, allowing other coroutines to be activated and move forward. This is a form of cooperative multitasking: coroutines voluntarily and explicitly yield control to the central scheduler. In contrast, threads implement preemptive multitasking. The scheduler can suspend threads at any time—even halfway through a statement—to give way to other threads.



### Three Styles of Using Generator
* Guido van Rossum wrote there are three different styles of code you can write using generators:
  * There’s the traditional “pull” style (iterators), “push” style (like the averaging example), and then there are “tasks” (Have you read Dave Beazley’s coroutines tutorial yet?…).
  * The David Beazley tutorial Guido refers to is “A Curious Course on Coroutines and Concurrency”.


## Four States of Coroutine
* A coroutine can be in one of four states. You can determine the current state using the `inspect.getgeneratorstate(…)` function, which returns one of these strings:
  * 'GEN_CREATED': Waiting to start execution.
  * 'GEN_RUNNING': Currently being executed by the interpreter. You’ll only see this state in a multithreaded application—or if the generator object calls getgeneratorstate on itself, which is not useful.
  * 'GEN_SUSPENDED': Currently suspended at a yield expression.
  * 'GEN_CLOSED': Execution has completed.

* Because the argument to the send method will become the value of the pending `yield` expression, it follows that you can only make a call like `my_coro.send(42)` if the coroutine is currently suspended. But that’s not the case if the coroutine has never been activated—when its state is 'GEN_CREATED'. That’s why the first activation of a coroutine is always done with `next(my_coro)`—you can also call `my_coro.send(None)`, and the effect is the same.

  
* It’s crucial to understand that the execution of the coroutine is suspended exactly at the yield keyword. As mentioned before, in an assignment statement, the code to the right of the = is evaluated before the actual assignment happens. This means that in a line like b = yield a, the value of b will only be set when the coroutine is activated later by the client code. It takes some effort to get used to this fact, but understanding it is essential to make sense of the use of yield in asynchronous programming. 

## Decorators for Coroutine Priming
* The initial call `next(my_coro)` is often described as “priming” the coroutine (i.e., advancing it to the first yield to make it ready for use as a live coroutine).
* Priming a coroutine before use is a necessary but easy-to-forget chore. To avoid it, a special decorator can be applied to the coroutine.

```
#coroutil.py: decorator for priming coroutine

from functools import wraps
def coroutine(func):
    """Decorator: primes `func` by advancing to first `yield`"""
    @wraps(func)
    def primer(*args,**kwargs):
        gen = func(*args,**kwargs)
        next(gen)
        return gen
    return primer
```

* The `yield from` syntax primes the coroutine called by it, making it incompatible with decorators such as `@coroutine`. The `asyncio.coroutine` decorator is designed to work with `yield form` so it does not prime the coroutine.

## Coroutine Termination and Exception Handling
* An unhandled exception within a coroutine propagates to the caller of the `next` or `send` that triggered it. Because the exception was not handled in the coroutine, it terminated. Any attempt to reactivate it will raise StopIteration.

* One way of terminating coroutines: you can use send with some sentinel value that tells the coroutine to exit. 
  * Constant built-in singletons like None and Ellipsis are convenient sentinel values. Ellipsis has the advantage of being quite unusual in data streams. 
  * Another sentinel value I’ve seen used is StopIteration—the class itself, not an instance of it (and not raising it). In other words, using it like: my_coro.send(StopIteration).
  
* Generator objects have two methods that allow the client to explicitly send exceptions into the coroutine—throw and close.
* If it’s necessary that some cleanup code is run no matter how the coroutine ends, you need to wrap the relevant part of the coroutine body in a try/finally block.

## Returning a Value from a Coroutine
* The value of the return expression in a coroutine is smuggled to the caller as an attribute of the `StopIteration` exception. This is a bit of a hack, but it preserves the existing behavior of generator objects: raising `StopIteration` when exhausted.  
  * One of the main reasons why the `yield from` construct was added to Python 3.3 has to do with throwing exceptions into nested coroutines. The other reason was to enable coroutines to return values more conveniently. 
  * `yield from` construct handles it automatically by catching StopIteration internally.
  * This is analogous to the use of StopIteration in for loops: the exception is handled by the loop machinery in a way that is transparent to the user. In the case of yield from, the interpreter not only consumes the StopIteration, but its value attribute becomes the value of the yield from expression itself. Unfortunately we can’t test this interactively in the console, because it’s a syntax error to use yield from—or yield, for that matter—outside of a function.

### Using yield from
* The first thing to know about yield from is that it is a completely new language construct. It does so much more than yield that the reuse of that keyword is arguably misleading. Similar constructs in other languages are called await, and that is a much better name because it conveys a crucial point: when a generator gen calls yield from subgen(), the subgen takes over and will yield values to the caller of gen; the caller will in effect drive subgen directly. Meanwhile gen will be blocked, waiting until subgen terminates.

* First, yield from can be used as a shortcut to yield in a for loop.

* The first thing the yield from x expression does with the x object is to call iter(x) to obtain an iterator from it. This means that x can be any iterable.
* The real nature of yield from cannot be demonstrated with simple iterables; it requires the mind-expanding use of nested generators. That’s why PEP 380, which introduced yield from, is titled “Syntax for Delegating to a Subgenerator.”
  * The main feature of yield from is to open a bidirectional channel from the outermost caller to the innermost subgenerator, so that values can be sent and yielded back and forth directly from them, and exceptions can be thrown all the way in without adding a lot of exception handling boilerplate code in the intermediate coroutines. This is what enables coroutine delegation in a way that was not possible before.

* If a subgenerator never terminates, the delegating generator will be suspended forever at the `yield from`. This will not prevent your program from making progress because the yield from (like the simple yield) transfers control to the client code (i.e., the caller of the delegating generator). But it does mean that some task will be left unfinished.
* Every yield from chain must be driven by a client that calls next(…) or .send(…) on the outermost delegating generator. This call may be implicit, such as a for loop.

* First Approximation Descrition of yield from
  * When the iterator is another generator, the effect is the same as if the body of the subgenerator were inlined at the point of the yield from expression. Furthermore, the subgenerator is allowed to execute a return statement with a value, and that value becomes the value of the yield from expression.

### Terminologies
* delegating generator: The generator function that contains the yield from <iterable> expression.
* subgenerator: The generator obtained from the <iterable> part of the yield from expression. This is the “subgenerator” mentioned in the title of PEP 380: “Syntax for Delegating to a Subgenerator.”
* caller: PEP 380 uses the term “caller” to refer to the client code that calls the delegating generator. Depending on context, I use “client” instead of “caller,” to distinguish from the delegating generator, which is also a “caller” (it calls the subgenerator).
 
 













## Generators as Coroutines
* Like .__next__(), .send() causes the generator to advance to the next yield, but it also allows the client using the generator to send data into it: whatever argument is passed to .send() becomes the value of the corresponding yield expression inside the generator function body. In other words, .send() allows two-way data exchange between the client code and the generator—in contrast with .__next__(), which only lets the client receive data from the generator.

* This is such a major “enhancement” that it actually changes the nature of generators: when used in this way, they become coroutines. David Beazley—probably the most prolific writer and speaker about coroutines in the Python community—warned in a famous PyCon US 2009 tutorial:
  * Generators produce data for iteration
  * Coroutines are consumers of data
  * To keep your brain from exploding, you don’t mix the two concepts together
  * Coroutines are not related to iteration
  * Note: There is a use of having `yield` produce a value in a coroutine, but it’s not tied to iteration.
— David Beazley “A Curious Course on Coroutines and Concurrency”


## Further Reading
* David Beazley is the ultimate authority on Python generators and coroutines.
* Beazley’s PyCon tutorials on the subject are legendary for their depth and breadth. The first was at PyCon US 2008: “Generator Tricks for Systems Programmers”. PyCon US 2009 saw the legendary “[A Curious Course on Coroutines and Concurrency](http://www.dabeaz.com/coroutines/)” (hard-to-find video links for all three parts: [part 1](http://pyvideo.org/video/213), [part 2](http://pyvideo.org/video/215), [part 3](http://pyvideo.org/video/214)). His most recent tutorial from PyCon 2014 in Montréal was “Generators: The Final Frontier,” in which he tackles more concurrency examples

* Coroutines allow new ways of organizing code, and just as recursion or polymorphism (dynamic dispatch), it takes some time getting used to their possibilities. An interesting example of classic algorithm rewritten with coroutines is in the post “Greedy algorithm with coroutines,” by James Powell. You may also want to browse “Popular recipes tagged coroutine" in the ActiveState Code recipes database.

Paul Sokolovsky implemented yield from in Damien George’s super lean MicroPython interpreter designed to run on microcontrollers. As he studied the feature, he created a great, detailed diagram to explain how yield from works, and shared it in the python-tulip mailing list. Sokolovsky was kind enough to allow me to copy the PDF to this book’s site, where it has a more permanent URL.

* Paul Sokolovsky implemented yield from in Damien George’s super lean MicroPython interpreter designed to run on microcontrollers. As he studied the feature, he created a [great, detailed diagram](http://bit.ly/1JIqGxW) to explain how yield from works, and shared it in the python-tulip mailing list. Sokolovsky was kind enough to allow me to copy the PDF to this book’s site, where it has a [more permanent URL](http://flupy.org/resources/yield-from.pdf).

* Greg Ewing—who penned PEP 380 and implemented yield from in CPython—published [a few examples](http://bit.ly/1JIqJtu) of its use: a BinaryTree class, a simple XML parser, and a task scheduler.

* Other interesting examples of yield from without asyncio appear in a message to the Python Tutor list, “[Comparing two CSV files using Python](http://bit.ly/1JIqSxf)” by Peter Otten, and a Rock-Paper-Scissors game in Ian Ward’s “[Iterables, Iterators, and Generators](http://bit.ly/1JIqQ8x)” tutorial published as an iPython notebook.

* Guido van Rossum sent a long message to the python-tulip Google Group titled “[The difference between yield and yield-from](http://bit.ly/1JIqT44)" that is worth reading. Nick Coghlan posted a heavily commented version of the yield from expansion to [Python-Dev on March 21, 2009](http://bit.ly/1JIqRcv);

* Experimenting with discrete event simulations is a great way to become comfortable with cooperative multitasking. Wikipedia’s “[Discrete event simulation](http://bit.ly/1JIqXB1)” article is a good place to start.[145] A short tutorial about writing discrete event simulations by hand (no special libraries) is Ashish Gupta’s “[Writing a Discrete Event Simulation: Ten Easy Lessons.](http://bit.ly/1JIqWgz)” The code is in Java so it’s class-based and uses no coroutines, but can easily be ported to Python. Regardless of the code, the tutorial is a good short introduction to the terminology and components of a discrete event simulation. Converting Gupta’s examples to Python classes and then to classes leveraging coroutines is a good exercise.





* In programming languages, keywords establish the basic rules of control flow and expression evaluation. A keyword in a language is like a piece in a board game.
* Adding a new piece to Chess would be a radical change. Adding a new keyword in a programming language is also a radical change. So it makes sense for language designers to be wary of introducing keywords.

```
Keywords	Language	Comment
5

Smalltalk-80

Famous for its minimalist syntax.

25

Go

The language, not the game.

32

C

That’s ANSI C. C99 has 37 keywords, C11 has 44.

33

Python

Python 2.7 has 31 keywords; Python 1.5 had 28.

41

Ruby

Keywords may be used as identifiers (e.g., class is also a method name).

49

Java

As in C, the names of the primitive types (char, float, etc.) are reserved.

60

JavaScript

Includes all keywords from Java 1.0, many of which are unused.

65

PHP

Since PHP 5.3, seven keywords were introduced, including goto, trait, and yield.

85

C++

According to cppreference.com, C++11 added 10 keywords to the existing 75.

555

COBOL

I did not make this up. See this IBM ILE COBOL manual.

∞

Scheme

Anyone can define new keywords.

```
* Python 3 added nonlocal, promoted None, True, and False to keyword status, and dropped print and exec. It’s very uncommon for a language to drop keywords as it evolves.
  * The most serious manifestation of this problem is the overloading of def: it’s now used to define functions, generators, and coroutines—objects that are too different to share the same declaration syntax.
  * A highly recommended post related to this issue in the context of JavaScript, Python, and other languages is “[What Color Is Your Function?](http://bit.ly/1JIrIdh)” by Bob Nystrom.
  
* Scheme inherited from Lisp a macro facility that allows anyone to create special forms adding new control structures and evaluation rules to the language. The user-defined identifiers of those forms are called “syntactic keywords.” The Scheme R5RS standard states “There are no reserved identifiers” (page 45 of the standard), but a typical implementation such as MIT/GNU Scheme comes with 34 syntactic keywords predefined, such as if, lambda, and define-syntax—the keyword that lets you conjure new keywords.
