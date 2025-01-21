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
## Distributed tx

https://en.wikipedia.org/wiki/X/Open_XA

### Msg broker + DB 

#### Outbox pattern

https://www.decodable.co/blog/revisiting-the-outbox-pattern

https://youtube.com/watch?v=sJYzmFfCL2Y

https://www.atomikos.com/Documentation/Resources
