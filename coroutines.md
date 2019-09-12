# Table of Contents

*[](#)
*[](#)
*[](#)


## Generators as Coroutines
* Like .__next__(), .send() causes the generator to advance to the next yield, but it also allows the client using the generator to send data into it: whatever argument is passed to .send() becomes the value of the corresponding yield expression inside the generator function body. In other words, .send() allows two-way data exchange between the client code and the generator—in contrast with .__next__(), which only lets the client receive data from the generator.

* This is such a major “enhancement” that it actually changes the nature of generators: when used in this way, they become coroutines. David Beazley—probably the most prolific writer and speaker about coroutines in the Python community—warned in a famous PyCon US 2009 tutorial:
  * Generators produce data for iteration
  * Coroutines are consumers of data
  * To keep your brain from exploding, you don’t mix the two concepts together
  * Coroutines are not related to iteration
  * Note: There is a use of having `yield` produce a value in a coroutine, but it’s not tied to iteration.
— David Beazley “A Curious Course on Coroutines and Concurrency”
