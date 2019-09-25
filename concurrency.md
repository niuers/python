# Table of Contents

* [Concurrency with Futures](#concurrency-with-futures)
  * [Futures Definition](#futures-definition)
    * [Characteristics](#characteristics)
    * [Two Futures](#two-futures)
  * [Blocking I/O and the GIL](#blocking-io-and-the-gil)
  * [concurrent.futures](#concurrent-futures)
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
* The concurrent.futures examples are limited by the GIL.
* The GIL is nearly harmless with I/O-bound processing.
* The CPython interpreter is not thread-safe internally, so it has a Global Interpreter Lock (GIL), which allows only one thread at a time to execute Python bytecodes. That’s why a single Python process usually cannot use multiple CPU cores at the same time.
  * This is a limitation of the CPython interpreter, not of the Python language itself. Jython and IronPython are not limited in this way; but Pypy, the fastest Python interpreter available, also has a GIL.

* When we write Python code, we have no control over the GIL, but a built-in function or an extension written in C can release the GIL while running time-consuming tasks. In fact, a Python library coded in C can manage the GIL, launch its own OS threads, and take advantage of all available CPU cores. This complicates the code of the library considerably, and **most library authors don’t do it**.
  * `sleep` always releases the GIL.
* However, all standard library functions that perform blocking I/O release the GIL when waiting for a result from the OS. This means Python programs that are I/O bound can benefit from using threads at the Python level: while one Python thread is waiting for a response from the network, the blocked I/O function releases the GIL so another thread can run. The time.sleep() function also releases the GIL.
* That’s why David Beazley says: “Python threads are great at doing nothing.”
* Therefore, Python threads are perfectly usable in I/O-bound applications, despite the GIL.
* Python threads are well suited for I/O-intensive applications, and the concurrent.futures package makes them trivially simple to use for certain use cases.

## concurrent.futures
* The main features of the `concurrent.futures` package are the `ThreadPoolExecutor` and `ProcessPoolExecutor` classes, which implement an interface that allows you to submit callables for execution in different threads or processes, respectively. The classes manage an internal pool of worker threads or processes, and a queue of tasks to be executed. 

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
  
## Future Reading
* 











