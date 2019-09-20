# Table of Contents

* [](#)
* [](#)
* [](#)


# Concurrency with Futures
## Futures Definition
> Futures: objects representing the asynchronous execution of an operation.

### Characteristics
* Futures encapsulate pending operations so that they can be put in queues, their state of completion can be queried, and their results (or exceptions) can be retrieved when available.
* An important thing to know about futures in general is that you and I should not create them: they are meant to be instantiated exclusively by the concurrency framework, be it concurrent.futures or asyncio.
  * For example, the Executor.submit() method takes a callable, schedules it to run, and returns a future.

### Two Futures
* There are two classes named Future in the standard library: concurrent.futures.Future and asyncio.Future. 
* They serve the same purpose: an instance of either Future class represents a deferred computation that may or may not have completed.     * This is similar to the Deferred class in Twisted, the Future class in Tornado, and Promise objects in various JavaScript libraries.
* Both types of Future have a .done() method that is nonblocking and returns a Boolean that tells you whether the callable linked to that future has executed or not. Instead of asking whether a future is done, client code usually asks to be notified. That’s why both Future classes have an .add_done_callback() method: you give it a callable, and the callable will be invoked with the future as the single argument when the future is done.

* There is also a .result() method, which works the same in both classes when the future is done: it returns the result of the callable, or re-raises whatever exception might have been thrown when the callable was executed. However, when the future is not done, the behavior of the result method is very different between the two flavors of Future. 
  * In a concurrency.futures.Future instance, invoking f.result() will block the caller’s thread until the result is ready. An optional timeout argument can be passed, and if the future is not done in the specified time, a TimeoutError exception is raised. 
  * The asyncio.Future.result method does not support timeout, and the preferred way to get the result of futures in that library is to use yield from—which doesn’t work with concurrency.futures.Future instances.
  
## Blocking I/O and the GIL
* The GIL is nearly harmless with I/O-bound processing.
* The CPython interpreter is not thread-safe internally, so it has a Global Interpreter Lock (GIL), which allows only one thread at a time to execute Python bytecodes. That’s why a single Python process usually cannot use multiple CPU cores at the same time.
  * This is a limitation of the CPython interpreter, not of the Python language itself. Jython and IronPython are not limited in this way; but Pypy, the fastest Python interpreter available, also has a GIL.

* When we write Python code, we have no control over the GIL, but a built-in function or an extension written in C can release the GIL while running time-consuming tasks. In fact, a Python library coded in C can manage the GIL, launch its own OS threads, and take advantage of all available CPU cores. This complicates the code of the library considerably, and **most library authors don’t do it**.
* However, all standard library functions that perform blocking I/O release the GIL when waiting for a result from the OS. This means Python programs that are I/O bound can benefit from using threads at the Python level: while one Python thread is waiting for a response from the network, the blocked I/O function releases the GIL so another thread can run. The time.sleep() function also releases the GIL.
* That’s why David Beazley says: “Python threads are great at doing nothing.”
* Therefore, Python threads are perfectly usable in I/O-bound applications, despite the GIL.

## Launching Processes with concurrent.futures
* A simple way to work around the GIL for CPU-bound jobs using concurrent.futures is to use the ProcessPoolExecutor class to bypass the GIL and leveraging all available CPU cores, if you need to do CPU-bound processing.
* If you are doing CPU-intensive work in Python, you should try PyPy.
















