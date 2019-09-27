# Table of Contents

* [Concurrency with Futures](#concurrency-with-futures)
  * [Futures Definition](#futures-definition)
    * [Characteristics](#characteristics)
    * [Two Futures](#two-futures)
  * [Blocking I/O and the GIL](#blocking-io-and-the-gil)
  * [concurrent.futures](#concurrentfutures)
* [Concurrency with asyncio](#concurrency-with-asyncio)



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
* The `concurrent.futures` examples are limited by the GIL.
* The GIL is nearly harmless with I/O-bound processing.
  * The CPython interpreter is not thread-safe internally, so it has a Global Interpreter Lock (GIL), which allows only one thread at a time to execute Python bytecodes. That’s why a single Python process usually cannot use multiple CPU cores at the same time.
  * This is a limitation of the CPython interpreter, not of the Python language itself. Jython and IronPython are not limited in this way; but Pypy, the fastest Python interpreter available, also has a GIL.
  * When we write Python code, we have no control over the GIL, but a built-in function or an extension written in C can release the GIL while running time-consuming tasks. In fact, a Python library coded in C can manage the GIL, launch its own OS threads, and take advantage of all available CPU cores. This complicates the code of the library considerably, and **most library authors don’t do it**.
  * However, all standard library functions that perform blocking I/O release the GIL when waiting for a result from the OS. 
    * This means Python programs that are I/O bound can benefit from using threads at the Python level: while one Python thread is waiting (blocked) for a response from the network, the blocked I/O function releases the GIL so another thread can run. 
    * The `time.sleep()` function always releases the GIL.
  * That’s why David Beazley says: “Python threads are great at doing nothing.”
  * Therefore, Python threads are perfectly usable in I/O-bound applications, despite the GIL.
  * Python threads are well suited for I/O-intensive applications, and the concurrent.futures package makes them trivially simple to use for certain use cases.

## concurrent.futures
* The main features of the `concurrent.futures` package are the `ThreadPoolExecutor` and `ProcessPoolExecutor` classes, which implement an interface that allows you to submit callables for execution in different threads or processes, respectively. The classes manage an internal pool of worker threads or processes, and a queue of tasks to be executed. 
* concurrent.futures package is exciting: it treats threads, processes, and queues as infrastructure at your service, not something you have to deal with directly. Of course, it’s designed with simple jobs in mind, the so-called [“embarrassingly parallel”](https://en.wikipedia.org/wiki/Embarrassingly_parallel) problems.

### Launching Threads with `concurrent.futures`
* Python threads are well suited for I/O-bound applications, despite the GIL: every standard library I/O function written in C releases the GIL, so while a given thread is waiting for I/O, the Python scheduler can switch to another thread.

#### Using `Executor.map`
* The `Executor.map` function is easy to use but it has a feature that may or may not be helpful, depending on your needs. 
  * It returns the results exactly in the same order as the calls are started: if the first call takes 10s to produce a result, and the others take 1s each, your code will block for 10s as it tries to retrieve the first result of the generator returned by map. After that, you’ll get the remaining results without blocking because they will be done. 

#### Using `Executor.submit` and `futures.as_completed`
* Often it’s preferable to get the results as they are ready, regardless of the order they were submitted. 
  * To do that, you need a combination of the `Executor.submit` method and the `futures.as_completed` function.
  * The combination of `executor.submit` and `futures.as_completed` is more flexible than `executor.map` because you can submit different callables and arguments, while `executor.map` is designed to run the same callable on the different arguments. In addition, the set of futures you pass to `futures.as_completed` may come from more than one executor—perhaps some were created by a `ThreadPoolExecutor` instance while others are from a `ProcessPoolExecutor`.
  * Calling the `result` method on a future either returns the value returned by the callable, or raises whatever exception was caught when the callable was executed. This method may block waiting for a resolution, but not in this example because as_completed only returns futures that are done.
  * This example uses an idiom that’s very useful with `futures.as_completed`: building a dict to map each future to other data that may be useful when the future is completed. Here the `to_do_map` maps each future to the country code assigned to it. This makes it easy to do follow-up processing with the result of the futures, despite the fact that they are produced out of order.

```
    with futures.ThreadPoolExecutor(max_workers=concur_req) as executor:
        to_do_map = {}
        for cc in sorted(cc_list):
            future = executor.submit(download_one,
                            cc, base_url, verbose)
            to_do_map[future] = cc 
        done_iter = futures.as_completed(to_do_map) 
        if not verbose:
            done_iter = tqdm.tqdm(done_iter, total=len(cc_list)) 
        for future in done_iter:
            try:
                res = future.result()
            except requests.exceptions.HTTPError as exc: 
                error_msg = 'HTTP {res.status_code} - {res.reason}'
                error_msg = error_msg.format(res=exc.response)
            except requests.exceptions.ConnectionError as exc:
                error_msg = 'Connection error'
            else:
                error_msg = ''
                status = res.status
```

### Launching Processes with `concurrent.futures`
* The `concurrent.futures` documentation page is subtitled “Launching parallel tasks”. The package does enable truly parallel computations because it supports distributing work among multiple Python processes using the `ProcessPoolExecutor` class—thus bypassing the GIL and leveraging all available CPU cores, if you need to do CPU-bound processing.
* If you are doing CPU-intensive work in Python, you should try PyPy.

## Threading and Multiprocessing Alternatives

* If `futures.ThreadPoolExecutor` is not flexible enough for a certain job, you may need to build your own solution out of basic threading components such as Thread, Lock, Semaphore, etc.—possibly using the thread-safe `queue`s of the queue module for passing data between threads. Those moving parts are encapsulated by `futures.ThreadPoolExecutor`.

* For CPU-bound work, you need to sidestep the GIL by launching multiple processes. 
  * The `futures.ProcessPoolExecutor` is the easiest way to do it. But again, if your use case is complex, you’ll need more advanced tools. The `multiprocessing` package emulates the threading API but delegates jobs to multiple processes. For simple programs, multiprocessing can replace threading with few changes. But multiprocessing also offers facilities to solve the biggest challenge faced by collaborating processes: how to pass around data.
  
## Other Languages
* Go, Elixir, and Clojure provide better, higher-level, concurrency abstractions, as the Seven Concurrency Models book demonstrates. Erlang—the implementation language of Elixir—is a prime example of a language designed from the ground up with concurrency in mind.
* Like Lisp and Clojure, Elixir implements syntactic macros. That’s a double-edged sword. Syntactic macros enable powerful DSLs, but the proliferation of sublanguages can lead to incompatible codebases and community fragmentation. Lisp drowned in a flood of macros, with each Lisp shop using its own arcane dialect. Standardizing around Common Lisp resulted in a bloated language. I hope José Valim can inspire the Elixir community to avoid a similar outcome.

* Go is a modern language with fresh ideas. But, in some regards, it’s a conservative language, compared to Elixir. Go doesn’t have macros, and its syntax is simpler than Python’s. Go doesn’t support inheritance or operator overloading, and it offers fewer opportunities for metaprogramming than Python. These limitations are considered features. They lead to more predictable behavior and performance. That’s a big plus in the highly concurrent, mission-critical settings where Go aims to replace C++, Java, and Python.
* But in the history of programming languages, the conservative ones tend to attract more coders. I’d like to become fluent in Go and Elixir.

* Concurrency in the Competition
  * MRI—the reference implementation of Ruby—also has a GIL, so its threads are under the same limitations as Python’s. Meanwhile, JavaScript interpreters don’t support user-level threads at all; asynchronous programming with callbacks is their only path to concurrency.

# Concurrency with asyncio

## Concurrency vs. Parallelism
* Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once. Not the same, but related. One is about structure, one is about execution. 
* Concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable. By Rob Pike, Co-inventor of the Go language, "Concurrency Is Not Parallelism (It’s Better)”.
* For real parallelism, you must have multiple cores. A modern laptop has four CPU cores but is routinely running more than 100 processes at any given time under normal, casual use. So, in practice, most processing happens concurrently and not in parallel. The computer is constantly dealing with 100+ processes, making sure each has an opportunity to make progress, even if the CPU itself can’t do more than four things at once. Ten years ago we used machines that were also able to handle 100 processes concurrently, but on a single core. That’s why Rob Pike titled that talk “Concurrency Is Not Parallelism (It’s Better).”

## Thread Versus Coroutine: A Comparison


## asyncio
* `asyncio` is a package that implements concurrency with coroutines driven by an event loop. 
* `asyncio` uses a stricter definition of “coroutine.” 
  * A coroutine suitable for use with the asyncio API must use `yield from` and not `yield` in its body. 
  * Also, an `asyncio` coroutine should be driven by a caller invoking it through `yield from` or by passing the coroutine to one of the asyncio functions such as `asyncio.async(…)` and others. 
  * Finally, the `@asyncio.coroutine` decorator should be applied to coroutines
  
* Never use time.sleep(…) in asyncio coroutines unless you want to block the main thread, therefore freezing the event loop and probably the whole application as well. If a coroutine needs to spend some time doing nothing, it should yield from asyncio.sleep(DELAY).

* An asyncio.Task is roughly the equivalent of a threading.Thread. Victor Stinner, special technical reviewer for this chapter, points out that “a Task is like a green thread in libraries that implement cooperative multitasking, such as gevent.”
  * A Task drives a coroutine, and a Thread invokes a callable.
  * When you get a Task object, it is already scheduled to run (e.g., by asyncio.async); a Thread instance must be explicitly told to run by calling its start method.
  * There’s no API to terminate a thread from the outside, because a thread could be interrupted at any point, leaving the system in an invalid state. For tasks, there is the Task.cancel() instance method, which raises CancelledError inside the coroutine. The coroutine can deal with this by catching the exception in the yield where it’s suspended.
  * you know how challenging it is to reason about the program because the scheduler can interrupt a thread at any time. You must remember to hold locks to protect the critical sections of your program, to avoid getting interrupted in the middle of a multistep operation—which could leave data in an invalid state.
  * With coroutines, everything is protected against interruption by default. You must explicitly yield to let the rest of the program run. Instead of holding locks to synchronize the operations of multiple threads, you have coroutines that are “synchronized” by definition: only one of them is running at any time. And when you want to give up control, you use yield or yield from to give control back to the scheduler. That’s why it is possible to safely cancel a coroutine: by definition, a coroutine can only be cancelled when it’s suspended at a yield point, so you can perform cleanup by handling the CancelledError exception.

### asyncio.Future: Nonblocking by Design
* The asyncio.Future and the concurrent.futures.Future classes have mostly the same interface, but are implemented differently and are not interchangeable.
* `futures` are created only as the result of scheduling something for execution. 
* In asyncio, BaseEventLoop.create_task(…) takes a coroutine, schedules it to run, and returns an asyncio.Task instance—which is also an instance of asyncio.Future because Task is a subclass of Future designed to wrap a coroutine. This is analogous to how we create concurrent.futures.Future instances by invoking Executor.submit(…).
* Using yield from with a future automatically takes care of waiting for it to finish, without blocking the event loop—because in asyncio, yield from is used to give control back to the event loop.

* Note that using yield from with a future is the coroutine equivalent of the functionality offered by add_done_callback: instead of triggering a callback, when the delayed operation is done, the event loop sets the result of the future, and the yield from expression produces a return value inside our suspended coroutine, allowing it to resume.

* In summary, because asyncio.Future is designed to work with yield from, these methods are often not needed:
  * You don’t need my_future.add_done_callback(…) because you can simply put whatever processing you would do after the future is done in the lines that follow yield from my_future in your coroutine. That’s the big advantage of having coroutines: functions that can be suspended and resumed.
  * You don’t need my_future.result() because the value of a yield from expression on a future is the result (e.g., result = yield from my_future).

### Yielding from Futures, Tasks, and Coroutines
* In asyncio, there is a close relationship between futures and coroutines because you can get the result of an asyncio.Future by yielding from it. This means that res = yield from foo() works if foo is a coroutine function (therefore it returns a coroutine object when called) or if foo is a plain function that returns a Future or Task instance. This is one of the reasons why coroutines and futures are interchangeable in many parts of the asyncio API.

* In order to execute, a coroutine must be scheduled, and then it’s wrapped in an asyncio.Task.

* If you want to experiment with futures and coroutines on the Python console or in small tests, you can use the following snippet:3
```
>>> import asyncio
>>> def run_sync(coro_or_future):
...     loop = asyncio.get_event_loop()
...     return loop.run_until_complete(coro_or_future)
...
>>> a = run_sync(some_coroutine())
```
* The relationship between coroutines, futures, and tasks is documented in section 18.5.3. Tasks and coroutines of the asyncio documentation, where you’ll find this note:

In this documentation, some methods are documented as coroutines, even if they are plain Python functions returning a Future. This is intentional to have a freedom of tweaking the implementation of these functions in the future.

### How to Understand the `yield from` code
* It is easy to follow the code with `yield from` if you employ a trick suggested by Guido van Rossum himself: squint and pretend the `yield from` keywords are not there. If you do that, you’ll notice that the code is as easy to read as plain old sequential code.
* Using the yield from foo syntax avoids blocking because the current coroutine is suspended (i.e., the delegating generator where the yield from code is), but the control flow goes back to the event loop, which can drive other coroutines. When the foo future or coroutine is done, it returns a result to the suspended coroutine, resuming it.

### The facts about every usage of yield from
* Every arrangement of coroutines chained with `yield from` must be ultimately driven by a caller that is not a coroutine, which invokes `next(…)` or `.send(…)` on the outermost delegating generator, explicitly or implicitly (e.g., in a for loop).
* The innermost subgenerator in the chain must be a simple generator that uses just yield—or an iterable object.

* When using yield from with the `asyncio` API, both facts remain true, with the following specifics:
  * The coroutine chains we write are always driven by passing our outermost delegating generator to an asyncio API call, such as `loop.run_until_complete(…)`.
  * In other words, when using `asyncio` our code doesn’t drive a coroutine chain by calling `next(…)` or `.send(…)` on it—the asyncio event loop does that.
  * The coroutine chains we write always end by delegating with `yield from` to some asyncio coroutine function or coroutine method (e.g., `yield from asyncio.sleep(…)`) or coroutines from libraries that implement higher-level protocols (e.g., `resp = yield from aiohttp.request('GET', url)` ).
  * In other words, the innermost subgenerator will be a library function that does the actual I/O, not something we write.

* To summarize: as we use `asyncio`, our asynchronous code consists of coroutines that are delegating generators driven by asyncio itself and that ultimately delegate to asyncio library coroutines—possibly by way of some third-party library such as `aiohttp`. This arrangement creates pipelines where the `asyncio` event loop drives—through our coroutines—the library functions that perform the low-level asynchronous I/O.

### Running Circles Around Blocking Calls
* There are two ways to prevent blocking calls to halt the progress of the entire application:
  * Run each blocking operation in a separate thread.
    * Threads work fine, but the memory overhead for each OS thread—**the kind that Python uses**—is on the order of megabytes, depending on the OS. We can’t afford one thread per connection if we are handling thousands of connections.
  * Turn every blocking operation into a nonblocking asynchronous call.
    * Callbacks are the traditional way to implement asynchronous calls with low memory overhead. They are a low-level concept, similar to the oldest and most primitive concurrency mechanism of all: hardware interrupts. Instead of waiting for a response, we register a function to be called when something happens. In this way, every call we make can be nonblocking. Ryan Dahl advocates callbacks for their simplicity and low overhead.
    * Of course, we can only make callbacks work because the event loop underlying our asynchronous applications can rely on infrastructure that uses interrupts, threads, polling, background processes, etc. to ensure that multiple concurrent requests make progress and they eventually get done. When the event loop gets a response, it calls back our code. But the single main thread shared by the event loop and our application code is never blocked—if we don’t make mistakes.
    * When used as coroutines, generators provide an alternative way to do asynchronous programming. From the perspective of the event loop, invoking a callback or calling .send() on a suspended coroutine is pretty much the same. There is a memory overhead for each suspended coroutine, but it’s orders of magnitude smaller than the overhead for each thread. And they avoid the dreaded “callback hell,” which we’ll discuss in “From Callbacks to Futures and Coroutines”.











  





## Future Reading
* For a modern take on concurrency without threads or callbacks, Seven Concurrency Models in Seven Weeks, by Paul Butcher (Pragmatic Bookshelf) is an excellent read. 
* If you are intrigued about the GIL, start with the Python Library and Extension FAQ ([“Can’t we get rid of the Global Interpreter Lock?”](https://docs.python.org/3/faq/library.html#id18)). Also worth reading are posts by Guido van Rossum and Jesse Noller (contributor of the multiprocessing package): “[It isn’t Easy to Remove the GIL](https://www.artima.com/weblogs/viewpost.jsp?thread=214235)” and “Python Threads and the Global Interpreter Lock.” Finally, David Beazley has a detailed exploration on the inner workings of the GIL: “[Understanding the Python GIL.](http://www.dabeaz.com/GIL/)” In [slide #54](http://www.dabeaz.com/python/UnderstandingGIL.pdf) of the presentation, Beazley reports some alarming results, including a 20× increase in processing time for a particular benchmark with the new GIL algorithm introduced in Python 3.2. However, Beazley apparently used an empty while True: pass to simulate CPU-bound work, and that is not realistic. The issue is not significant with real workloads, according to a comment by Antoine Pitrou—who implemented the new GIL algorithm—in the bug report submitted by Beazley.













