# Table of Contents

* [Context Managers and with Blocks](#context-managers-and-with-blocks)
* [](#)


# Context Managers and with Blocks
* The `with` statement sets up a temporary context and reliably tears it down, under the control of a context manager object. This prevents errors and reduces boilerplate code, making APIs at the same time safer and easier to use.
* Context manager objects exist to control a `with` statement, just like iterators exist to control a `for` statement.
* The `with` statement was designed to simplify the `try/finally` pattern, which guarantees that some operation is performed after a block of code, even if the block is aborted because of an exception, a return or `sys.exit()` call. The code in the `finally` clause usually releases a critical resource or restores some previous state that was temporarily changed.
* The context manager protocol consists of the `__enter__` and `__exit__` methods. At the start of the `with`, `__enter__` is invoked on the context manager object. The role of the `finally` clause is played by a call to `__exit__` on the context manager object at the end of the `with` block.
* The most common example is making sure a file object is closed.
* However, `with` blocks don’t define a new scope, as functions and modules do. So after a `with` statement, the variable is still available, e.g. a file.
* The context manager object is the result of evaluating the expression after `with`, but the value bound to the target variable (in the `as` clause) is the result of calling `__enter__` on the context manager object (i.e. returned from `__enter__`).
* When control flow exits the `with` block in any way, the `__exit__` method is invoked on the context manager object, not on whatever is returned by `__enter__`.

## On `__exit__`
* The interpreter calls the `__enter__` method with no arguments—beyond the implicit self. The three arguments passed to `__exit__` are:
  * exc_type
  * exc_value
  * traceback
* The three arguments received by `self` are exactly what you get if you call `sys.exc_info()` in the finally block of a `try/finally` statement. This makes sense, considering that the with statement is meant to replace most uses of try/finally, and calling `sys.exc_info()` was often necessary to determine what clean-up action would be required.

## Using @contextmanager
* @contextmanager decorator is also intriguing because it shows a use for the yield statement unrelated to iteration.
* instead of writing a whole class with __enter__/__exit__ methods, you just implement a generator with a single yield that should produce whatever you want the __enter__ method to return.
  * In a generator decorated with @contextmanager, yield is used to split the body of the function in two parts: everything before the yield will be executed at the beginning of the `with` block when the interpreter calls `__enter__`; the code after yield will run when `__exit__` is called at the end of the block.
  * Yield the value that will be bound to the target variable in the as clause of the with statement. This function pauses at this point while the body of the with executes.
* Essentially the contextlib.contextmanager decorator wraps the function in a class that implements the __enter__ and __exit__ methods.
* Recall that the __exit__ method tells the interpreter that it has handled the exception by returning True; in that case, the interpreter suppresses the exception. On the other hand, if __exit__ does not explicitly return a value, the interpreter gets the usual None, and propagates the exception. With @contextmanager, the default behavior is inverted: the __exit__ method provided by the decorator assumes any exception sent into the generator is handled and should be suppressed.[126] You must explicitly re-raise an exception in the decorated function if you don’t want @contextmanager to suppress it. This convention was adopted because when context managers were created, generators could not return values, only yield. They now can, as explained in Returning a Value from a Coroutine. As you’ll see, returning a value from a generator does involve an exception.
  * Having a try/finally (or a with block) around the yield is an unavoidable price of using @contextmanager, because you never know what the users of your context manager are going to do inside their with block.
* Note that the use of yield in a generator used with the @contextmanager decorator has nothing to do with iteration. In the examples shown in this section, the generator function is operating more like a coroutine: a procedure that runs up to a point, then suspends to let the client code run until the client wants the coroutine to proceed with its job.

* A key point that Raymond Hettinger made in his PyCon US 2013 keynote is that with is not just for resource management, but it’s a tool for factoring out common setup and teardown code, or any pair of operations that need to be done before and after another procedure (slide 21, What Makes Python Awesome?).


# `else` Blocks
* the else clause can be used not only in if statements but also in for, while, and try statements.

## The Rules of using else
* for: The else block will run only if and when the for loop runs to completion (i.e., not if the for is aborted with a break).
* while: The else block will run only if and when the while loop exits because the condition became falsy (i.e., not when the while is aborted with a break).
* try: The else block will only run if no exception is raised in the try block. The official docs also state: “Exceptions in the else clause are not handled by the preceding except clauses.”
* In all cases, the else clause is also skipped if an exception or a return, break, or continue statement causes control to jump out of the main block of the compound statement.

* Using else with these statements often makes the code easier to read and saves the trouble of setting up control flags or adding extra if statements.

### For/Else
* The general pattern of for/else
```
for item in my_list:
    if item.flavor == 'banana':
        break
else:
    raise ValueError('No banana flavor found!')
```

### Try/Else
* We need to convert following pattern
```
try:
    dangerous_call()
    after_call()
except OSError:
    log('OSError...')
```

To this one with `else. For clarity and correctness, the body of a try block should only have the statements that may generate the expected exceptions. This is much better:

```
try:
    dangerous_call()
except OSError:
    log('OSError...')
else:
    after_call()
```

* Now it’s clear that the try block is guarding against possible errors in dangerous_call() and not in after_call(). It’s also more obvious that after_call() will only execute if no exceptions are raised in the try block.
* The else-clause itself is interesting. It runs when there is no exception but before the finally-clause. That is its primary purpose.

In Python, try/except is commonly used for control flow, and not just for error handling. There’s even an acronym/slogan for that documented in the official Python glossary:
  * EAFP: Easier to ask for forgiveness than permission. This common Python coding style assumes the existence of valid keys or attributes and catches exceptions if the assumption proves false. This clean and fast style is characterized by the presence of many try and except statements. The technique contrasts with the LBYL style common to many other languages such as C.
  * LBYL: Look before you leap. This coding style explicitly tests for pre-conditions before making calls or lookups. This style contrasts with the EAFP approach and is characterized by the presence of many if statements. In a multi-threaded environment, the LBYL approach can risk introducing a race condition between “the looking” and “the leaping”. For example, the code, if key in mapping: return mapping[key] can fail if another thread removes key from mapping after the test, but before the lookup. This issue can be solved with locks or by using the EAFP approach.

* Regarding Pythonic usage of try/except, with or without else, Raymond Hettinger has a brilliant answer to the question “Is it a good practice to use try-except-else in Python?” in StackOverflow. 

## Further Reading
* Raymond Hettinger highlighted the with statement as a “winning language feature” in his PyCon US 2013 keynote. He also showed some interesting applications of context managers in his talk “Transforming Code into Beautiful, Idiomatic Python” at the same conference.
