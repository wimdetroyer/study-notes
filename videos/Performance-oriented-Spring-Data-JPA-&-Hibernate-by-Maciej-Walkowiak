Performance-oriented-Spring-Data-JPA-&-Hibernate-by-Maciej-Walkowiak

https://www.youtube.com/watch?v=L9ZOgX-3LTQ


# better metrics

- flexypool
- p6spy
- digma -> static analysis
- tracing (opentelemetry)

https://stackoverflow.com/a/23907711/3470438

# configuration tuning

by default we have **10 connections** in the pool 

## disable spring OSIV

- reminder: hibernate session is just EntityManager with some extra bells and whistles
- OSIV works by _initializing_ the session to the DB and keeping it open from the controller layer, note that a transaction is NOT acquired at the controller level.
- once the FULL request is complete OSIV closes the session: the DB connection is kept open even after the transactional block is closed! 


See also: 
https://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch02.html#session-per-operation
https://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch02.html#transaction-antipatterns
https://stackoverflow.com/a/69076110/3470438
https://stackoverflow.com/a/5640796/3470438
https://stackoverflow.com/a/48222934/3470438

## disable datasource autocommit

- side effect: if in a @transactional method a non-db blocking call (http request) is done _before_ the DB call, smart enougg to only actually open tx after first db interaction
- inverse is more difficult: use programmatic api instead of declarative for more fine grained control.
- disable on the db pool (but also at hibernate side?)


https://vladmihalcea.com/why-you-should-always-use-hibernate-connection-provider_disables_autocommit-for-resource-local-jpa-transactions/


## other gotchas

## transactional propagation requires new

-> itr will make a 2nd db connection, and it will keep 2 connections open if it is spawned from another exisitng trans...


## Registerbanktransferusecase example issues 

### spring jpa -> isNew() : unneccessary selects

If we pass a application generated ID, spring save() will return false on isNew() in its _naive_ implementation. Causing an unneccessary select in the db

- fix 1 use version: https://vladmihalcea.com/jpa-entity-version-property-hibernate/ 
useful if you are going to use _optimistic locking_ anyway: https://stackoverflow.com/questions/5751271/optimistic-vs-multi-version-concurrency-control-differences
- fix 2: implement persistable, and implement isNew method

### lack of transactionality: entities become unmanaged immediately after fetch
use @transactional.

### no need to fetch full entity getById

use getReferenceById instead, which does not actually fetch in the DB, but gives a shallow proxy reference so you can link it to your banktransfer object

alternative: don't use entities cross aggregrations, use ids

## SettleBanktransferUseCase

### Dont use fetchtype.eager by default

###  N+1 query problem

