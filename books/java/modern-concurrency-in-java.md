# Study notes & reflections - Modern Concurrency in Java: Virtual Threads, Structured Concurrency, and Beyond by A N M Bazlur Rahman

## Chapter 1 


## Chapter 2 - primer on VT

## ForkJoinPool as the performant base of carrier threads managing virtual threads


### Limitations of virtual threads

#### Pinning during synchronized blocks

Because the JVM has no way to know when a virtual thread can be _parked_ (correct term?) when a synchronized block is entered. Alleviated by using ReentrantLocks which use park/unparking instead of intrinsic lock mechanism (which?). Fixed in J24.

#### Pinning during native code calls.

Native code is out of control of the JVM, so virtual threads will remain thread to the platform thread for the duration. seems there is no solution for this. 
