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

### Definition of thread safety

It's hard to come up with a definition of thread safety.

a thread safe class however is a class which is _correct_. **correctness** is defined as a program/class behaving in accordance to its _specifications_.

It's easy for a class to be **correct** or **thread safe** when it is accessed by a single thread, so one could argue that if a class still functions according to its _specifications_ when it is accessed by **multiple** threads, it is **thread safe**.

#### Parts of the specification

##### Invariants

A class has certain _invariants_. These are properties that  cannot/ should not change _before_ and _after_ exeuction.
For example, a NumberRange class could have two state variables _lower_ and _upper_ which could be set in a _concurrent_ fashion, but with the invariant that the lower bound is always smaller than the upper bound.

PS: This is in contrast to a _precondition_ which is something that should be held true before a function is executed (which is easy to verify in single threaded programs, but can be influenced by another thread in concurrrent programs (see: wait/notify))

##### Postconditions

Something that should hold true _after_ a function is executed. 

If we have a thread safe _contains_ function on a list class, a postcondition of this function is that it will promise us to tell us that there is or is not an element in this list.
 
### Important part of thread safety: Atomicity

An atomic operation means an operation which must be done all at once. It is _indivisible_ in to different operations (A in ACID)


#### Problems with non atomic operations
non atomic operations examples:

- read, modify, write
- check, then act (put if absent)

##### Race conditions

These operations could cause **race conditions**.  (due to for example thread interleaving)

###### Example 1 - stale values
Thread A could read a stale value of a non thread safe variable B, and use it for further processing, while thread B updates it a little while later.

###### Example 2 - singleton pattern

Another example of this: a well known pattern is the singleton pattern to implement _lazy loading_ . Defer the construction of an expensive object until when it is needed, and then store the reference for further usage. With unlucky timing, thread A and B could 'check' the current reference, see it is both empty, and then both construct the expensive object.


#### Atomic variable building blocks in java.concurrent to the rescue

There are atomic variable helpers (AtomicReference, AtomicLong, ...)

For instance, it is easy to create a thread safe counter class by wrapping the counter in an AtomicLong and then using the thread safe incrementAndGet method.

**HOWEVER** if we have two atomic variables, the code is no longer thread safe because while the variables are **individually** thread safe, using them in combination provides no guarantees anymore. If multiple variables participate in an invariant, they are not _independent_ (The number range class is a good example of this, which has two state variables lower and upper which participate in its invariant)


### Locks

Java provides _intrinsic locks_ which are locks _built in_ to the object. They're also called _monitor locks_ (which is what shows up in the java bytecode).

It is done within the confines of a _synchronized_ block.

If a synchronized keyword is placed in the method signature, then this is shorthand for a synchronized block but wrapped to the entire method with the current object as the lock. We can also pass another reference as the lock.

These locks act as _mutexes_ or mutual exclusion locks: **At most one thread can hold the key to the lock** . if thread A currently has the lock on some operation Z, thread B must wait until thread A _releases_ this lock (and vice versa). Bad locking policies can cause _liveness_ issues as already mentioned (for instance deadlocking if thread A waits for B and thread B waits for A, eternally)



#### Re-entrancy

Locks are re-entrant: meaning that within the same thread, a method can be called multiple times, where a counter is incremented by 1 for each time an operation is pending within that thread.

For example, if we have a method foo and a thread A calls it 3 times, it will have to wait until the execution is done 3 times for the lock counter to be zero. then and only then will the method be opened up for access to another thread. This means locks are implemented on a **per thread** basis instead of on a **per invocation** basis.


## Chapter 3 - Sharing objects


### Important part of thread safety: VISIBILITY



## Chapter 4 - Composing objects

### Client side locking

Client side locking means that _client code_ using an object X uses the intrinsic lock object X has to guard its state.

It is not preferred to use client side locking by way of _extending_ the base class, because it breaks the encapsulation of the synchronization policy of the (thread safe) base class we are extending

So, just as you would prefer **composition over inheritance** to not ti**e implementation from the base class to the derived class**, do not tie** the synchronization policy of the base class to the derived class** by extending it.

#### Composition as an alternative

Prefer to use _composition_. 

1. Does not spread synchronization policy over multiple classes, encapsulation is not broken
2. clearer to extend, can use the lock of the implementing class

#### Examples
See [code samples](https://github.com/wimdetroyer/java-sandbox/blob/main/src/main/java/be/wimdetroyer/javasandbox/jcip/ImprovedList.java) for more info on 


### Documenting thread safe classes

A class' **synchronization policy** is an important part of your **program design**. **Document** it to describe how the access is governed to data/state in the face of concurrency.

Multiple areas which are important in describing your **synchronization policy**:

- the use of _volatile_ variables
- the use of locks to determine which variables are controlled with a lock
- which variables are _thread confined_ (remember ThreadLocal?)
- 


 
