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

A task is any  piece of code which can be structured as an **independent** unit of work. (download a file, make a http request, ..)
Because of it is independence, it lends itself to be executed in a thread by means of an executor.
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

## Chapter 7 - cancellation & shutdown

Java does not provide a _pre-emptive_ mechanism to stop threads. It does _exist_ but one should NEVER use it as it is unsafe (and deprecated)

Instead, a _cooperative_ mechanism is used. A thread is _interrupted_ and asked to stop what it is doing when it feels like it (the programmer is also free to ignore this). 
While this might seem less useful than providing a forcing mechanism, it actually gives control to the programmer in order to shut down a thread gracefully.

Why would you want to cancel a task or interrupt a thread?

- resource management
- time limited activities
- shutdown
- errors
- users requested cancellation
- ...


### Programming around interruption/cancellaton

See code samples for _thread_ and _scheduler_

https://github.com/wimdetroyer/java-sandbox/tree/main/src/main/java/be/wimdetroyer/javasandbox/jcip/interruption

**when possible, use the executor service to handle cancellation**. Moreover, the executor service has a _shutdown_ method which can be called and is _called automatically_ when using try-with-resources since **java 19** because it became auto-closeable (source: https://stackoverflow.com/a/72267126/3470438)



### JVM shutdown / daemon threads / finalizers

- JVM has _shutdown hooks_ which can be registered, which will get executed upon jvm shutdown. Be **very** careful when programming these. Avoid it.
- Daemon threads are background threads that **do not inhibit** the JVM from shutting down. Prefer to put non-blocking background tasks (periodic logging, cleanup of tmp files, ...) in here.
- Avoid the use of finalizers.

## Chapter 8 - Thread pools 'deep dive' (wink ;) )

This chapter is all about thread pools, how to use them, how to tune them.


### Caveats of the executor service: complete decoupling of task submission and execution was an oversimplification.

There are some caveats the previous chapter didn't mention about task execution.

If a task relies on the execution of another task, you risk liveness problems.

If a task uses threadlocal, it must die when the task is done, because an executorservice using a threadpool can reuse threads.

Tasks that deepends on other task can also pose deadlock problems. For instance, if two tasks are submitted to a single threaded executor, where task 1 is dependent on task two, the application will always deadlock. 

If it happens in a multithreaded executor (using a thread pool). you risk _thread starvation deadlock_ If you run out of threads, you're SOOL. 

In conclusion: Thread pools work best when tasks are homogeneous and independent. 

### sizing threadpools 

<img width="487" alt="image" src="https://github.com/user-attachments/assets/28ce3d19-9a34-4c07-b499-ad5fc2d244e3">

For CPU intensive tasks, a good ballpark is CPU cores + 1 (bcs even when all cpus are occupied, faults can happen).

For blocking tasks (webservers etc), use more threads, but in the advent of virtual threads, theres not much of a point anyway.

8.5 parallelization -> see example in code sandbox


### Thread pools & thread local

https://stackoverflow.com/questions/7403809/java-cached-thread-pool-and-thread-local

## Chapter 9 - GUI applications

No big take-aways here for me.

## Chapter 10 - Liveness hazards

As we know, liveness is the state of the program in which good things _eventually keep happening_. Liveness hazards cause this to no longer be the case.

### Deadlock

The primary example is deadlock. The classic example is the one of the dining philosophers. 

5 philosophers are seated at a round table, eating spaghetti (or taking a break from eating). You need two forks to eat the spaghetti, but theres only 5 forks.

So philosopher 1 starts eating the spaghetti by taking up the left and right fork, taking a bite, then putting it down. So far so good!

Now lets say all philosophers want to eat at the same time, they all try to take the fork on the left, that works! but then they try to  pick up the fork on the right, but that means they have to wait fork  the fork on the other philosophers' left to be available... but the other philosopher cannot put his fork back down because he is also waiting...

This example illustrates deadlock: Thread A is waiting on thread B to complete an action, but thread B needs to wait on thread A to complete its action.

## Chapter 11 - Performance

Avoid using concurrency when you can, because most of the time, YAGNI. Premature optimization is the root of all evil, after all.

If you _do_ consider taking advantage of threading, make sure to do it in a measured way:

- size of dataset
- by which resource is the operation _bound_ ? (cpu, memory, network, ...)
- is the loss of clarity and maintainability worth the potential (measure! if it does) speedup in performance?

### Amdahl's law

Programs consist of parts which can be executed and decomposed  _serially_ (sequentially, one at a time) and _concurrently_

Amdahl's law defines the speedup an application can have maximally by taking into account the ratio of serial and concurrent parts.
**For instance, if 50% of the program _has_ to be executed serially, than only a maximum speedup of 2x can be had (on the remaining 50% concurrent code)**

### Trade-offs when using threads

#### Context switching

When more runnable threads are used than available cpu cores, at a certain point, the CPU scheduler will _pre-empt_ one thread to stop running, **save it's current context**, and then _context switch_ to a new thread, loading in **a new context**
This _context switch_ is costly precisely because of this, and should be avoided.

## Chapter 12 - testing

### Testing for correctness
See code sample https://github.com/wimdetroyer/java-sandbox/tree/main/src/test/java/be/wimdetroyer/javasandbox/jcip/testing

have _sequential_ tests to make sure initial spec is correct, then introduce concurrent tests for the other behaviour.

Make sure tests 

- use randomized test data so the compiler cannot optimize
- have as much interleavings as possible in order to simulate concurrency better (one can do this with synchronizers such as cyclic barrier in order for all threads to start & stop at same time)
    - use thread.yield() to allow other threads some action
- test itself relies on as little concurrency as possible, because of the 'chicken and egg problem' (needing to test concurrent code implies that your test code is also concurrent)

#### Memory leaks

Test for memory leaks, however it is strange this is somehow particular to concurrency, not just in general...
One does this by doing heap dumps and inspecting. In the BoundedBuffer example, if we hadn't dereferenced popped elements, we'd have a memory leak.


instrumentation is the act of modifying software so that analysis can be performed on it, one can achieve it by decoration.



### Testing for performance
find a typical usage scenario, write a program that executes that scenario many times, and time it.


See code sample.


#### More resources

https://github.com/openjdk/jcstress/tree/master
https://www.youtube.com/watch?v=zI1RrlHZM8Y -> lincheck

## Chapter 13 - Explicit locks

Synchronized blocks a.k.a intrinsic locks are easy to grasp: it automatically opens and releases the lock for you in the same method.

However, sometimes one wants more finetuned control over synchronization. 

### Re-entrant lock
The _reentrantlock_ is a fine example of this. 

#### Negligeable advantages:

- define a thread fairness policy (YAGNI in most cases)
- lock can be _released_ after method/ block exit (but this is more difficult code to understand)
- it performs better in java 5, and negligeably(?) better after java 6 - not worth the switch
- locks don't necesseraliy have to block the thread. ReEntrantslock tryLock() is a non-blocking variant of this (more in chapter 15)

#### 'real' advantages:

timed acquisition and cancel acquisition: in contrast to the intrinsic locks, deadlock does not mean the 'end' of a thread, we can back off and try again (with a factor of randomness to prevent livelock), or cancel.


### Read-write lock

Lock logically divided into a read and write lock, useful for datastructures where there is _a lot of_ reading and _sparse_ writes.

## Chapter 14 - Building custom synchronizers

Skipped.

## Chapter 15 - Atomic variables & nonblocking synchronization

Synchronization as a technique is costly. The mutueal exclusion lock *blocks* other threads by letting them _sleep_ (costly because you have to wake up the thread afterwards, restoring the context) or by _spin locking_ (wasting cpu cycles). While the JVM can release locks fairly quickly initially, when _contention_ on a lock is increased when it is _held longer_ and _more often_ the JVM is forced to go to the OS, and this is costly.

It would be nice to have soem sort of a non-blocking algorithm, where a thread can query a method and simply say 'sorry, result is not ready for you, but try back another time'.

Atomic variables, under the hood, provide such a light-weight form of concurrency which is more performant, and it does so by utilizing _non-blocking_ algorithms.

The atomic variables implemented in the _java.util.concurrent_ package use CAS (Compare And Swap) in the JVM if possible, and if not, at the hardware level.

### Optimistic vs pessimistic locking

_Pessimistic_ locking assumes the worst: if i don't lock this variable away until i'm done, other threads could interfere with my work, so please wait until I am done.
_Optimistic_ locking is basically the 'it is easier to ask for forgiveness than ask for permission' principle. A thread could execute an operation while another thread is also doing the update, but via _collision detection_ we detect if an error was done and then rectify it. 

Compare and Swap is such example of optimistic locking implemented at the hardware level as a non-blocking atomic operation.

### Compare And Swap

Conceptually, Compare and swap is like the following: We have a memory location V. We want to set it to a _new_ value B, and we expect in V to be an _expected old_ value A. if A is not in V at the time we do this operation, nothing happens, if it is, the new value of B will be a. 

Equivalent: I think memory location X has the value 1 as the current old value, and i'd like to set it to 2 now. If X is not value 1 right now, don't do anything, but tell me i was wrong, if it was ok, make the update.

Code sample [here](https://github.com/wimdetroyer/java-sandbox/blob/main/src/main/java/be/wimdetroyer/javasandbox/jcip/cas/SimulatedCAS.java)

### Atomic variables

Atomic variables are _better volatile variables_. The most interesting are the _scalars_

- atomic integer
- atomic reference
- atomic long
- atomic booleaan

which support CAS.

#### A better number range, with no locking, using atomic ref

see https://github.com/wimdetroyer/java-sandbox/tree/main/src/main/java/be/wimdetroyer/javasandbox/jcip/numberrange


#### Performance trade-offs for locks vs atomic variables: pseudo random number genrator example

Use atomics for low to moderate contention, and locks for high contention, but if you can, **avoid shared state** all together!

### non blocking datastructures

examples:
- https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ConcurrentLinkedQueue.html
- https://lmax-exchange.github.io/disruptor/disruptor.html

Let the smart people do the research into difficult non-blocking algorithms, and then reap the rewards by using these performant datastructures in your code.

## Chapter 16 - The Java memory model

The JMM (Java Memory Model) is a low-level topic we haven't covered yet. Ideally, developers should just focus on proper synchronization an  safe publication.
That being said, it is always **useful** to know the details, and you can even _piggyback_ on existing synchronization (obviating the need for locking, by example) because of the _guarantees_ the JMM gives on _synchronized code_ allowing to squeeze even further performance benefits out of concurrent code.


### Reorderings

As we know, the JVM and hardware (CPU) will _reorder_ statements. These reorderings are the driving reason of a lot of performance improvements.

For example, the JVM can - within a single thread -, the JVM can reorder any statement, as long as the program executes the same way as if it would if we the statements were executed in _program order_. (something known as _within thread as if serial_ semantic)

Once multiple threads kick in, the JVM needs to define a set of _guarantees_ in order for it to still play nice.

Because reordering is so **omnipresent** and **frequent**, it is _very_ hard to reason about if you don't use proper _synchronization_ to inhibit it. 

> [!NOTE]
> Note that this is not the same as _interleaving_ . A thread is said to _interleave_ with another thread if its code is executed in between the execution of the code of the other thread


### The JMM is defined by actions and the _happens before_ guarantee

Actions are:

- reads and writes to a variable
- unlocking and locking of a lock/monitor
- starting and joining of threads

A _partial ordering_ (y prefer sushi to cheeseburgers and Mozart to Mahler, but we don’t
necessarily have a clear preference between cheeseburgers and Mozart) is defined on these actions. the 'happens before' guarantee.

Examples:

'if a thread A acquires a lock on an object, makes some writes to some state of this object, and then releases the lock, when a thread B interacts with said class (again, by acquiring a new lock), the changes done by thread A will be guaranteed to have _happened before_ the execution of thread B. 

Other examples include:
- Each action WITHIN a single thread happens-before every action in that thread that comes later in the program's order. (as alrdy mentioned)
- writes of volatile variables happens before any read of said volatile variable

See: https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility)

#### Data races

A _data race(s)_ between two (or more) threads is defined as two (or more) accesses (of which atleast one write) to a variable not ordered by a happens-before relationsip.

Two accesses to (reads of or writes to) the same variable are said to be conflicting if at least one of the accesses is a write.

When a program contains two conflicting accesses that are not ordered by a happens-before relationship, it is said to contain a data race.

A program is said to be _correctly synchronized_ if it does not contain any data races.


It is not to be confused with a race condition, which was covered earlier

### Piggybacking of the happens before guarantee

As mentioned, we can piggy back of earlier synchronization to benefit from the _happens before guarantee_ enabled by it.

Higher level constructs such as the building blocks in java.util.concurrent also follow this logic:

- you always know that in a blocking queue, _putting_ an element happens _before_ taking it
- a barrier action always happens _after_ the barrier is broken
- ...

### Safe initialization

The lazy initialization holder class idiom is shown here (which is also present in effective java)
The double-checked locking pattern is described as an anti-pattern. Noted.


Further watching: https://www.youtube.com/watch?v=qADk_tj4wY8


Fin.

--------------------------
# Running of on a tangent...

## Appendix 1 - where JCIP became somewhat 'dated' when java evolved along

See: [history](https://medium.com/kaustav-das/the-evolution-of-multi-threading-capabilities-in-java-e6aa24dd01e6)

## Java 8 
### Fork/Join framework

ForkJoinPool as the base of this framework, an implementation of the ExecutorService.

https://www.baeldung.com/java-fork-join#forkJoinPool
https://stackoverflow.com/questions/3524634/java-7-fork-join-framework
https://medium.com/@satyendra.jaiswal/mastering-parallelism-with-the-fork-join-framework-in-java-3bb20970d716#:~:text=Fork%2DJoin%20leverages%20thread%20locality,cache%2C%20optimizing%20memory%20access%20patterns.
https://softwareengineering.stackexchange.com/questions/342232/what-is-the-difference-between-local-and-non-local-concurrency#:~:text=Local%20concurrency%20is%20defined%20as,%2C%20a%20web%20services%20interface).

**REPLACES** somewhat the concept of _work stealing_ in deques. 


### Parallel utilities

A good video by Brian Goetz on how you should change your thinking when you go from **sequential** to **parallel** : https://www.youtube.com/watch?v=NsDE7E8sIdQ

#### How? Divide and conquer / recursive decomposition

1. divide it into subproblems
2. solve the subproblems
3. combine the result

#### When to use? it depends.

- Is the problem even decomposable into subproblems which lends themselves to parallelism?
- do the costs of using parallelism outweigh the benefits?
   - splitting costs
   - task management costs
   - combine solutions costs (merging maps is expensive, for instance)
- Locality, for example there is a BIG difference between using primitive int array vs boxed integers because of the overhead of having to reference the objects. project valhalla aims to fix this with value types
- encounter order: some datastructures have an order (list) and some dont (set) and this can also influence performance


#### Parallel streams


**Builds upon the Fork/Join framework!**

https://www.baeldung.com/java-when-to-use-parallel-stream

Keep in mind this is not _magic performance dust_ . Use parallelism when it makes sense. (TODO: link vid of Brian Goetz and discuss key concepts)

#### Parallel sorting

Arrays.parallelSort()

### CompletableFuture

part of async / reactive progremming, imho no point to track

## Java 9

### Reactive programming (java.util.concurrent.flow)

IMHO no point learning this when virtual threads are a thing now

## Java 17

### Virtual Threads

https://www.youtube.com/watch?v=tykrCxwmMG4
https://github.com/victorrentea/java-latest/tree/vjug24
https://drive.google.com/file/d/19mFIC5fsmVzb3fBV0_aIH04uc7qZ_Yk-/view

* use them for I/O bound waiting operations (network, file , ...)
* not much gain for CPU intensive operations in comparison to platform threads
     * it can even be **detrimental** for you!
* not much need anymore for thread pooling?

#### Thread pinning

https://github.com/cescoffier/loom-unit

What are pinned threads?

Virtual threads are 'mounted' to a _carrier thread_ ( / platform thread) . after running some code and awaiting execution of blocking operations, the VT can _unpin_ itself from the carrier thread.

However, when  a VT does a blocking operation when

- a native method or foreign function is called
- the VT is executing a synchronized block

the VT is said to be _pinned_ to the carrier thread, meaning it cannot be unmounted. This can hinder scalability.

Wim De Troyer
​​You mentioned the 'danger' of enabling virtual threads in spring. how can we configure it safely ?

Simone Bordet
​​@Wim De Troyer you cannot. You have an if/else branch in a library that pins your carrier, you now have a problem you may not discover beforehand

##### Use ReentrantLocks instead of Synchronized

https://todd.ginsberg.com/post/java/virtual-thread-pinning/#avoid
https://stackoverflow.com/a/11821900/3470438
https://stackoverflow.com/questions/78671922/why-reentrantlock-is-better-for-virtual-threads-than-synchronized

=> Solution is being worked on, see: https://www.reddit.com/r/java/comments/1d578af/new_loom_ea_builds_with_changes_to_object_monitor/?share_id=KQzZk0qvElLIRht9V5WtO

### ScopedValue as a replacement of ThreadLocal (preview) !

[here](https://www.baeldung.com/java-20-scoped-values)

## Onwards and upwards...

### Structured Concurrency

- structured concurrency _policies_ (eg: success & failure) an alternative of chapter 7?


https://www.reddit.com/r/java/comments/1fazdkl/structuredtaskscope_vs_parallel_stream/


### Concurrent gatherers

https://softwaregarden.dev/en/posts/new-java/gatherers/concurrent/

## Appendix 2 - Random questions that popped up in my head while i was studying concurrency

### Relationship between threads and CPU cores. 

https://stackoverflow.com/questions/34689709/java-threads-and-number-of-cores
https://stackoverflow.com/questions/3126154/multithreading-what-is-the-point-of-more-threads-than-cores
https://stackoverflow.com/a/68796545/3470438

### How does the CPU schedule threads?

https://stackoverflow.com/questions/63912452/difference-between-time-slice-context-switch-and-thread-interference

If we have 4 CPU Cores, we can have 4 threads at maximum running in _parallel_ . However, this does not inhibit us from using more than four threads. Indeed, if we have 20 threads, any 4 of them will run parallel (4 CPU cores) and the 20 threads in total will be _scheduled_ by the CPU and each given some time, running _concurrenctly_ with possible _interleaved_ operations.


## appendix 3 - writing multithreaded code

...

## appendix 4 - parallels with other areas of computer science

- https://vladmihalcea.com/optimistic-vs-pessimistic-locking (udp : optimistic, accept loss vs tcp: pessimistic, retry)
- https://vladmihalcea.com/serializability/
- https://vladmihalcea.com/linearizability/

## Appendix 5 - the actor model instead of the OOP model for dealing with concurrent programs

- https://doc.akka.io/docs/akka/current/typed/guide/actors-motivation.html
- https://stackoverflow.com/questions/78318131/do-java-21-virtual-threads-address-the-main-reason-to-switch-to-reactive-single


### And what about concurrency in other programming paradigms entirely?


## Appendix 6 - more learning...

https://cs.lth.se/outdated/sde/phd-courses/advanced-concurrent-programming-in-java/

### Distributed locking? quid concurrency when not in the same machine?

https://dzone.com/articles/distributed-java-locks-with-redis


## Appendix 7 - All about policies, baby

synchronization policy, (task ) execution policy, task cancellation policy, thread interupption policy, saturation policy

Throughout the book, the concept of _policies_ is introduced. Here's an overview of every policy and a brief explanation

| **Policy**                   | **Overview**                                                                                                                                                                  |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Synchronization Policy**    | Defines how a program is _synchronized_ under concurrent execution. Examples include documenting which classes are thread-safe and explaining which state is governed by which lock (e.g., using `@ThreadSafe` annotations). A proper synchronization policy ensures _correctness_. |
| **Task Execution Policy**     | Defines primarily the _how_ (e.g., via thread pools) and _when_ (e.g., scheduling) of task execution. It also covers how task load should be shared across threads (e.g., via work stealing). |
| **Task Cancellation Policy**  | Defines how tasks should _cancel_. Should they cancel gracefully and clean up after themselves? Should they stop immediately? This also includes considerations like shutdown hooks and daemon threads. |
| **Thread Interruption Policy**| As Java uses a _pre-emptive_ thread stopping mechanism, this policy defines what a thread should do when interrupted: should it ignore, acknowledge and re-interrupt, or propagate the interruption? |
| **Saturation Policy**         | Governs how the program should degrade under heavy load. Should new tasks be ignored, or should the oldest tasks be discarded? An elegant strategy like the "Caller Executes" policy ensures tasks cascade appropriately through layers. |

