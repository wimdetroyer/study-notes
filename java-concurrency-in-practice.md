# JCIP

## Chapter 1 - intro

### Definitions

#### Liveness & safetyness

- Safetyness is the state a program has in which **bad things do not happen**
- Liveness is the state a program has in which eventually **good things keep happening**

Safetyness & liveness is trivial or irrelevant for single threaded applications, but when concurrent programming is introduced liveness and safetyness _failures_ can occur.

Example of a  safetyness failure: a counter class containing a counter property which is not thread-safe: a thread A can read a _stale_ value because the state chjange made in thread B is not _visible_ to thread A.
Example of a  liveness failure: A program _deadlocking_ because thread A waits until thread B fulfills an _action_, but in turn thread B is waiting for thread A to fullfill another _action_.

## Chapter 2 - Thread safety


Writing thread safe code in essence is writing code that provides access to _shared, mutable state_ in a safe way. _Shared mutable state_ being state that is visible and is able to be _mutated_ by more than one thread.

If **more than one** thread has access to some shared state, and _at least one of them writes to it at some point_ , the acesss to that shared state must be managed by use of _synchronization_.

it is important to write code with thread safety in mind, even though concurrency might not be needed at the time, because it is far easier to work with **existing** thread-safe code, than to **retrofit** it.

## Chapter 3 - TODO

## Chapter 4 - Composing objects

### Client side locking

Client side locking means that _client code_ using an object X uses the intrinsic lock object X has to guard its state.

When using client side locking, use _composition_.

See code samples for more info
