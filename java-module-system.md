# The Java Module System by Nicolai Parlog

## Chapter 1

### Common issues because of lack of modularity in artifacts/jars

#### Shadowing

##### Definition

Shadowing occurs when two (or more!) jars have a class with the same FQCN, and the class loader loads **the first one it can find.** The first one loaded in is then always used at run-time and it is said to **shadow** the other classes.

There is no determined way to know in which order the class-loader loads in JARs, so this behaviour can be unpredictable.

##### Example 1: multiple versions of the same library

Say library A has a `com.example.list.Iterator` class which was present in version 1.0.0 of the library but changed in version 2.0.0 of the library. 

If version 1.0.0 was loaded in first, it will use the Iterator present in 1.0.0. And the same goes for loading in version 2.0.0 first...


##### Example 2: shared class with same FQCN in multiple libraries.

if library A has a `com.example.list.Iterator` class and  library B has a `com.example.list.Iterator` class, the first one to be found by the class-loader will be used.


##### A way to overcome it

https://www.baeldung.com/maven-enforcer-plugin#1-ban-duplicate-dependency 
