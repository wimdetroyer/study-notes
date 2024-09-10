# JCIP
[[toc]]
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

Synchronization is the policy a program has to define safe access to a _critical section_ of a program: making sure it cannot be entered by more than one thread at the same time.

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

If a synchronized keyword is placed in the method signature, then this is shorthand for a synchronized block but wrapped to the entire method with the **current object as the lock**. We can also pass another reference as the lock.

These locks act as _mutexes_ or mutual exclusion locks: **At most one thread can hold the key to the lock** . if thread A currently has the lock because it is busy doing executing a method Z, thread B must wait until thread A _releases_ this lock (and vice versa). Bad locking policies can cause _liveness_ issues as already mentioned (for instance deadlocking if thread A waits for B and thread B waits for A, eternally)



#### Re-entrancy

Locks are re-entrant: meaning that if a thread holds a lock, within the same thread, methods can be called multiple times, where a counter is incremented by 1 for each time an operation is pending within that thread.

For example, if we have a method foo and a thread A calls it 3 times, it will have to wait until the execution is done 3 times for the lock counter to be zero. then and only then will the object be opened up for access to another thread. This means locks are implemented on a **per thread** basis instead of on a **per invocation** basis.


## Chapter 3 - Sharing objects

While chapter 2 talked about the techniques available to organize threading towards data in a safe way (synchronization), this chapter is about techniques for **structuring**  & **publishing** the **data** so it can be accessed in a thread-safe way.
 
Indeed, synchronization is not only about atomicity or demarcating critical sections, it is also about **memory visibility**. Making sure that changes effected upon some piece of state X by one thread are visible to _other_ threads.

### Important part of thread safety: (memory) VISIBILITY

In a single threaded environment, it is easy to reason about the state changes of a variable. As it is only set by one thread, the changes are _linear_ and follow the sequence of the code. When two or more threads _share_ access to some data, this is thrown out of the window: changes on data might not yet be _visible_ to another thread.

Moreover, synchronization ensures the following:

Say a thread A:

1. accesses a synchronized method B
2. thus acquiring a lock on it
3. make some changes to a state variable X
4. release the lock

then thread B

1. upon entering synchronized method B (note that the lock has been released)
2. will see the changes made to state variable X when thread A was executing method B **AND** all changes that thread A did before that
3. provided thread B uses the same lock as thread A used

Without synchronization, we don't have such a _happens before_ guaranteee

### the _volatile_ keyword: a weaker form of synchronization

While _synchronization_ ensures _atomicity_ of the instructions done in its block AND _memory visibility_ the volatile keyword only ensures _memory visibility_.

We can define a variable as volatile (for instance the int counter example) and this will ensure that **every change done upon this variable by one thread Ais immediately visible to another thread B**

In other words, we'll always see the most recent change, because we've told the compiler: hey, this value is shared, don't do any caching or reordering of instructions on this variable.

When to use the volatile keyword:

- use it on a variable acting as a completion/ status flag: ie variables that don't care about what the previous value is that it held for an update
- the variable is not part of an invariant involving other variables
- locking is not required when accesing the variable or setting it (non atomic)
  
The authors don't suggest to rely too heavily on this. It's harder to reason about.

### Publication of objects

Publishing an object means making it available for use from **outside of it's current scope**

We should take care when **publishing**

- publishing internal state variables of a class can break _encapsulation_ (instead of for instance governing the access to this variable via a getter)
- publishing objects which are not fully initialized / constructed can break _thread safety_

if an object is _published_ when it should not have been or not in time, it is said to have _escaped_.

publishing can happen:

directly -> making a state variable public 
indirectly -> a state variable containing a list of elements X, and a new element Yis added to this list X. Y is indirectly published because it is available thru the list wrapper reference.

Objects can _leak_ with this, and for instance if we have a list reference, instead of passing the list, we can instead pass a deep copy of this list

Publishing an inner class instance outwards will cause the outer class to be published, because an inner class has a reference to its outer class.

#### Don't: create and start a thread in the constructor

This can cause the thread to have access to the object when it is still in an incomplete unconstructed state. Instead, use a factory method and do your business after the construction has completed.

### Thread confinement

Thread confinement is the practise of _confining_ a variable to a thread: it is _only_ visible to this thread and this implies ofcourse that it is thread safe as the data is not _shared_ between threads.

#### Examples
- the ThreadLocal<?> wrapper, which wraps a reference to an object inside this class and assures that each accessing thread creates and maintains a copy of this variable for its lifecycle.
- the Connections in a connection pool
- using _volatile_ correctly: confining writes to a volatile variable to one thread, and allowing _multiple_ threads to read the value (remember that the volatile keyword causes the most recent state to be memory visible)

#### Stack confinement

Each thread has it's own _stack_ when doing operations: the things in those _stack_ are only visible to the thread. This means that if you have _local variables_ ie those which are for instance _scoped_ only within the method, you can safely use these variables.
Primitive types can even be returned from the method: as they do not have a _reference_ they cannot _escape_.

### Immutability

An object which is **immutable** is ALWAYS thread-safe.

An object is immutable if:

- it has the _final_ keyword
- if it is a collection (ie list) holding references to the underlying objects, those objects should be thread-safe / immutable too : (unmodifiableList)
- proper construction

 Using the _final_ keyword ensures _initizialiation safety_ which makes it so that the variable can be safely shared within threads.

An object is _effectively immutable_ if it is not _technically_ immutable, but the contents will not be changed after publication.                                                                                                                                       
 
## Chapter 4 - Composing objects

### Client side locking

Client side locking means that _client code_ using an object X uses the intrinsic lock object X has to guard its state.

It is not preferred to use client side locking by way of _extending_ the base class, because it breaks the encapsulation of the synchronization policy of the (thread safe) base class we are extending

So, just as you would prefer **composition over inheritance** to not ti**e implementation from the base class to the derived class**, do not tie **the synchronization policy of the base class to the derived class** by extending it.

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


## Chapter 5 - building blocks

the platform libraries provide some thread-safe building blocks we can use

### Synchronized collections

The Collections provides wrapper methods such as:

- Collections.synchronizedList()
- set ...

To wrap an existing list in a synchronized variant.
A synchronized collection ensures that all public methods are synchronized: this also implies that ALWAYS only max 1 thread at a time can access the collections' state & methods....

####  Problems

##### synchronized lists don't mean that compound actions somehow become thread safe

Indeed, if we have a some client code implementing a put-if-absent method on a synchronized list, we need to also synchronize this new method, giving it the same lock that is used for the synchronized list.
Sometimes it's not _clear_ which lock to use.

##### Performance impacts

This implies for instance that when we _iterate_ over a list, a thread holds a lock over the entire list, causing it to be unuseable for other threads for the duration of the iteration...

Prefer instead of synchronized collections to use _concurrent_ collections instead, which have performance wins.

### Concurrent collections

In contrast to synchronized collecitons which limit thread access on a collection wide lock, concurrent collections are built to allow multiple threads to safely access the collection.
They also define some commonly used compound actions on their API and assure the atomicity of calling them: example put-if-absent on the ConcurrentMap.

#### Concurrent hash map
ConcurrentHashMap uses lock striping

#### Copy on write array list 
 copies the entire list on every write, every read is therefore thread safe because it is effectively immutable

#### Blocking queues

Blocking queues (such as: ArrayBlockingQueue, LinkedListBlockingQueueu, PriorityBlockingQueues) are useful to implement the _producer-consumer_ pattern.
1. a _producer_ writes to the queue. if it is **unbound** the queue keeps growing (until you run out of memory ;-) ) . If it is _bound_ the producer will _block_ until the queue has place again, and then when it does, the item will be offered to the queue...
2. a _consumer_ consumes messages from the queue. If the queue is empty, the consumer will _block_ until a message is available.


#### Double Ended Queues: Deques

Whereas a queue only allows to peek from one end of the heap, a _deque_ allows insertion an deletion from the _head_ and _tail_ of the deque.
This pattern is useful to implement _work stealing_.
In contrast to a _producer - consumer_ pattern. Work stealing allows consumers to each have a reference of their own deque, and - should they run out of work - they are allowed to _steal_ work from the tail end (to reduce contention) of the deque of another consumer. 
This can improve performance because there is no _contention_ on a shared queue and less time is spent waiting.

 > [!NOTE]
>Contention is simply when two threads try to access either the same resource or related resources in such a way that at least one of the contending threads runs more slowly than it would if the other thread(s) were not running

### Thread states, blocking, interrupts, ...

![thread states](https://www.baeldung.com/wp-content/uploads/2021/04/Threadlifecycle.jpg)
(credits baeldung https://www.baeldung.com/java-interrupted-exception)

Threads can block/pause:

- waiting for to acquire a held lock
- awaking from Thread.sleep
- waiting for a result from another thread
- blocking on a blocking computation / I/O

**The thread will be put in a blocked state**

A thread in the _runnable_ state is available for pick-up.

A thread can be _interrupted_. When we _interrupt_ a thread, we tell the thread: stop what oyu're doing right now (waiting, running, ...) and go do something else. It's up to programmer to decide how the thread should contrinue

#### The InterruptedException

An interruptedException is a checked exception which can be thrown by a _blocking_ method to indicate that it is a blocking method and that it will stop early when its work is _interrupted_

##### What to do when you get one

A. Propagate it up to the calling method
B. handle it by, catching it, then setting the _interrupted_ flag on the _current thread_ as to not swallow the fact that the undelying thread was interrupted

**NEVER** catch the interrupted exception and do nothing!

### Synchronizers

A synchronizer is an object which coordinates the _control flow_ of a thread (waiting, running, ... ) based upon its state.

Examples:
- queues (wait until no longer full, wait until elem is in queue)
- latches
- semafores
-  barriers
-  ...

#### Latches

A _latch_ is a synchronizer that blocks threads until its  _terminal state_ is reached. Once it is reached, all threads are free to pass.
An example is a latch which waits until all players are connected in a multi-player game,  and then opening the latch so it can proceed.

##### Countdownlatch
An example of a latch is the CountDownLatch. The latch is initialized with a positive number, and on completion of any event necessary to unlock the latch, the countdown is decremented. Once the counter reaches zero, the threads are unblocked.

See [code sample](https://github.com/wimdetroyer/java-sandbox/blob/main/src/main/java/be/wimdetroyer/javasandbox/jcip/latch/CountDownLatchExample.java) for more info

##### FutureTask

Callable is an alternative to Runnable, specifying a result type which will be returned after the function has ran.

A _callable_ (or _runnable_ !) can be passed to a futuretask. A futureTask is the base implementation of a Future: an abstraction for the result of an async operation with state (for wait, complete, retrieve)
Cancellation is also supported.

See  [code sample](https://github.com/wimdetroyer/java-sandbox/blob/1d2c4c9fcacb85da3e7328785869dca7421508e9/src/main/java/be/wimdetroyer/javasandbox/jcip/futuretask) here

Futuretasks are handy because we can wrap them in a thread, run them, and then block by calling .get() until execution finishes up, gaining us some time :-)

##### Semaphores

https://stackoverflow.com/questions/12267096/blockingqueue-vs-semaphore

### Exercise: Building an efficient scalable concurrent result cache

See  [code sample](https://github.com/wimdetroyer/java-sandbox/tree/main/src/main/java/be/wimdetroyer/javasandbox/jcip/futuretask/efficientconcurrentcache) here for my attempt (more modern than the example in the book)

- concurrent hashmap for more performant concurrency (in lieu of synchronized)
- using FutureTask to handle edge case where two threads could be doing the same calculation at the same time
- atomic putifabsent

## Chapter 6 - Task execution

### Explanation of what a task is

TODO

#### ways to structure a web server

1. Sequential processing : a request comes in, and the server blocks until the request has been fulfilled
2. thread-per-request model w/ tasks created in thread: a request comes in, and a thread is started for each _task_$
3. thread-per-request model w/ tasks created in executor service: a request comes in, and the _executorservice_ handles creation, execution, resource handling, ... of the task

Obviously 3 is the more maintainable and performant solution.

### Executor abstraction

#### Benefits of using an executor service

- Using an executor decouples _task submission_ from _task execution_. 
- lifecycle support
- hooks (for instance for statistics
- Creation of threads is costly (less the case now with _virtual_ threads) so an executor can be made with a (cached) _thread pool_ which can reuse threads or set a cap on it, so we don't run out of memory.


#### Execution policy

An execution policy defines how, when (timing), what, where   a _task_ will be executed

- In which thread
- for how long?
- when (every day?)
- in what order? (fifo, lifo)
    - which tasks should be dropped when the system is overloaded?

 ### How to parallelize tasks

 **Heterogeneous** (ie tasks which all do something different) are bad candidates for parallelization, seeing as it is difficult to reconcile the calculations. Instead, try to parallelize _homogeneous_ tasks.
 Example: an HTML renderer which renders:
 - text
 - images

has better chances of performance increase if we paralellize the img fetch instead of parallezing text & image rendering.

# Chapter 7 - cancellation & shutdown

TODO

# Chapter 8 - Thread pools 'deep dive' (wink ;) )


## Thread pools & thread local

https://stackoverflow.com/questions/7403809/java-cached-thread-pool-and-thread-local

# Chapter 9 - GUI applications

No big take-aways here for me.


# Appendix - where JCIP became outdated

See: [history](https://medium.com/kaustav-das/the-evolution-of-multi-threading-capabilities-in-java-e6aa24dd01e6)

## Java 8 
### Fork/Join framework

ForkJoinPool as the base of this framework, an implementation of the ExecutorService.

https://www.baeldung.com/java-fork-join#forkJoinPool

https://stackoverflow.com/questions/3524634/java-7-fork-join-framework

**REPLACES** somewhat the concept of _work stealing_ in deques. 

**Builds upon the Fork/Join framework!**

#### Parallel streams

https://www.baeldung.com/java-when-to-use-parallel-stream

#### Parallel sorting

Arrays.parallelSort()

### CompletableFuture

## Java 9

### Reactive programming (java.util.concurrent.flow)

IMHO no point learning this when virtual threads are a thing now

## Java 17

### Virtual Threads

* use them for I/O bound waiting operations (network, file , ...)
* not much gain for CPU intensive operations in comparison to platform threads
 
### ScopedValue as a replacement of ThreadLocal (preview) !

[here](https://www.baeldung.com/java-20-scoped-values)

## Onwards and upwards...

### Structured Concurrency



