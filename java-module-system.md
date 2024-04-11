# The Java Module System by Nicolai Parlog

## Chapter 1

### The problem: lack of modularity in artifacts/jars - 'JAR HELL'


#### problem: Shadowing

##### Definition

Shadowing occurs when two (or more!) jars have a class with the same FQCN, and the class loader loads **the first one it can find.** The first one loaded in is then always used at run-time and it is said to **shadow** the other classes.

There is no determined way to know in which order the class-loader loads in JARs, so this behaviour can be unpredictable.

##### Example 1: multiple versions of the same library

Say library A has a `com.example.list.Iterator` class which was present in version 1.0.0 of the library but changed in version 2.0.0 of the library. 

If version 1.0.0 was loaded in first, it will use the Iterator present in 1.0.0. And the same goes for loading in version 2.0.0 first...


##### Example 2: shared class with same FQCN in multiple libraries.

if library A has a `com.example.list.Iterator` class and  library B has a `com.example.list.Iterator` class, the first one to be found by the class-loader will be used.


##### A way to overcome it
Build tools already default to using the most recent version by default.

Maven enforcer plugin can enforce not having more than one version:

https://www.baeldung.com/maven-enforcer-plugin#1-ban-duplicate-dependency 

#### problem: JARs exist in a 'big ball of mud' in the classpath: there is no concept of encapsulation

- classes in packages deemed for internal use can be easily accessed
- an example of this is `sun.misc.unsafe` used by big projects but intended to only be used internally!

#### Misc

- slow startup times because of class loading JAR lookup w/ linear scan
- 'all-or-nothing' for the java platform -> projects that don't need certain java JARs were bundled within the JDK anyway

### The solution: the JPMS (Java Platform Module System)

#### Definition of a module

A module consists of
- a name: _a module descriptor_
- its dependencies upon other modules
- its API (the packages it exports)

This is described in a _module declaration file_ a.k.a. the module-info.java file. Example:

```java
module be.wimdetroyer.foo {
  requires be.wimdetroyer.bar
  exports be.wimdetroyer.api
}
```

#### Lifecycle of the JPMS

##### Step 1 - bootstrapping

The JPMS bootstraps itself. The module system is _code_ so it is defined somewhere. The code is defined in the `java.base` **base module**.
The java module system and the base module bootstrap themselves... :-)

#### Step 2 - Module resolution & verification

The class loader looks up the modules residing in JARs. If a module is found, it is added to the module graph.
As mentioned, modules have **dependencies on other modules**. If the class loader cannot find the module in any JAR, the application will exit with an error.

#### Step 3 - launching the _initial modules'_ main method.

the JPMS launches the main method of the _initial module_ . The initial module is the place where Step 2 began its work. It shouldn't be confused with the base module which is part of the JDK.

The initial module must also contain the psvm method! via the module descriptor it can then be looked up.

#### Step 4 - the JPMS enforcing boundaries at runtime

when module A wants to access a type (class, ...) from module B, the JPMS checks:

- if said type is _public_
- if module B _exports_ that type
- if module A is connected to module B in the module graph (if module A _requires_ module B)

#### Goals of the JPMS

##### Reliable configuration

Dependencies can be found missing at _launch time_ instead of at _run time_ now (see step 4 of the lifecycle of the JPMS).

##### Stronger encapsulation

Only public members of public types of  _exported_ packages of a module are accessible from another module. If it does not satisfy all these constraints, this member is _inaccessible_ (**even when using reflection!**)

Sidenote:

>This also applies to the JDK, which, as described in the previous section, was turned
into modules. As a consequence, the module system prevents access to JDK-internal
APIs, meaning packages starting with sun. or com.sun.. Unfortunately, many widely
used frameworks and libraries like Spring, Hibernate, and Mockito use such internal
APIs, so many applications would break on Java 9 if the module system were that strict.
To give developers time to migrate, Java is more lenient: the compiler and JVM have
command-line switches that allow access to internal APIs; and, on Java 9 to 11, run-time
access is allowed by default 


Stronger encapsulation also increases **security** because it forbids access to internal jdk modules.


