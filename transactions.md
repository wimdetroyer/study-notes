https://www.marcobehler.com/guides/spring-transaction-management-transactional-in-depth

datasource vs transaction manager

how does spring transactional reconcile a rabbit transaction towards a db transaction ? 


transaction isolation levels

https://www.postgresql.org/docs/9.5/transaction-iso.html

isolation levels avoid _phenomena_


What are jdbc savepoints?

https://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html#set_roll_back_savepoints


https://www.atomikos.com/Main/WebHome

https://spring.io/blog/2011/08/15/configuring-spring-and-jta-without-full-java-ee


Hibernate version implements _optimistic locking_ howver ootb db also configures isolation elvels.

Hibernate first level cache is an example of application level repeartable reads: https://stackoverflow.com/questions/35068324/why-should-i-need-application-level-repeatable-reads

## Concurrency control

### Pessimistic locking

#### transaction isolation levels

https://www.postgresql.org/docs/9.5/transaction-iso.html

isolation levels avoid _phenomena_

### Optimistic locking

#### MVCC (multiversion concucrency control)

Hibernate impl.

Works completely differen than isolation levels (which is pessimistic locking)
 
https://stackoverflow.com/questions/41751192/isolation-level-vs-optimistic-locking-hibernate-jpa
https://stackoverflow.com/questions/10119787/optimistic-locking-in-hibernate-by-default
https://www.youtube.com/watch?v=2qXJI7kG1ig

## Distributed tx

- https://en.wikipedia.org/wiki/X/Open_XA
- saga pattern
- https://zhenyanghua.github.io/2019/04/distributed-transaction-management-with-rest-and-try-confirm-cancel-pattern/
- outbox
- 

### Msg broker + DB 

#### Outbox pattern

https://www.decodable.co/blog/revisiting-the-outbox-pattern

https://youtube.com/watch?v=sJYzmFfCL2Y

https://www.atomikos.com/Documentation/Resources

### What is the difference between saga pattern & XA/2PC ? 

https://stackoverflow.com/questions/48906817/2pc-vs-sagas-distributed-transactions


 ### HTTP Context / REST - is saga pattern the only option?

 https://www.atomikos.com/Blog/TransactionsForTheRESTOfUs

The fundamental challenge is that HTTP as a protocol doesn't natively support the core requirements of 2PC:

Lack of "prepare" semantics: HTTP calls don't have a standardized way to say "prepare but don't commit yet"
Statelessness: HTTP is inherently stateless, whereas 2PC requires maintaining transaction state
No built-in transaction identifier: HTTP doesn't have native transaction context propagation

However, some approaches have tried to implement 2PC-like behavior over HTTP:

REST-AT (REST Atomic Transactions): An implementation that defines specific endpoints for transaction coordination (prepare, commit, rollback)
WS-Coordination and WS-AtomicTransaction: Web service specifications for coordinating distributed activities, including 2PC over SOAP (though less common with modern REST APIs)
Custom implementations: Where services explicitly expose prepare/commit/abort endpoints that a coordinator can call

These approaches require:

Services to explicitly implement transaction participation endpoints
A transaction coordinator service
Transaction context propagation between services
Careful timeout handling

The industry generally moved away from these approaches because:

They're complex to implement correctly
They create tight coupling between services
They can lead to resource locking across service boundaries
They're prone to partial failures that are difficult to recover from
