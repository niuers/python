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

* The first thing the yield from x expression does with the x object is to call iter(x) to obtain an iterator from it. This means that x can be any iterable.

* The real nature of yield from cannot be demonstrated with simple iterables; it requires the mind-expanding use of nested generators. That’s why PEP 380, which introduced yield from, is titled “Syntax for Delegating to a Subgenerator.”
* The main feature of yield from is to open a bidirectional channel from the outermost caller to the innermost subgenerator, so that values can be sent and yielded back and forth directly from them, and exceptions can be thrown all the way in without adding a lot of exception handling boilerplate code in the intermediate coroutines. This is what enables coroutine delegation in a way that was not possible before.















## Generators as Coroutines
* Like .__next__(), .send() causes the generator to advance to the next yield, but it also allows the client using the generator to send data into it: whatever argument is passed to .send() becomes the value of the corresponding yield expression inside the generator function body. In other words, .send() allows two-way data exchange between the client code and the generator—in contrast with .__next__(), which only lets the client receive data from the generator.

* This is such a major “enhancement” that it actually changes the nature of generators: when used in this way, they become coroutines. David Beazley—probably the most prolific writer and speaker about coroutines in the Python community—warned in a famous PyCon US 2009 tutorial:
  * Generators produce data for iteration
  * Coroutines are consumers of data
  * To keep your brain from exploding, you don’t mix the two concepts together
  * Coroutines are not related to iteration
  * Note: There is a use of having `yield` produce a value in a coroutine, but it’s not tied to iteration.
— David Beazley “A Curious Course on Coroutines and Concurrency”
