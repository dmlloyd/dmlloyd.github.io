---
layout: post
title: JBoss Marshalling 1.2.0 Released, Website Upgraded
categories: 
date: 2010-01-05 14:28:00
---
 Some of you may have noticed that JBoss Marshalling 1.2.0.GA has been present [in the maven repository](http://repository.jboss.org/maven2/org/jboss/marshalling/ "") for some time. Now that [the website](http://www.jboss.org/jbossmarshalling "") upgrade is complete, I'm happy to announce that it is now available from [the downloads page](http://www.jboss.org/jbossmarshalling/downloads "") as well.  
JBoss Marshalling is a framework which wholly replaces Java Serialization with a new API and optional optimized wire format. The standard wire format can also be used as well - giving compatibility with applications using standard serialization, but without the various bugs present therein, and with a potentially significant performance advantage.  
Other features that JBoss Marshalling provides which are missing from the Java Object*Stream API include:

* Pluggable class resolvers, making it easy to customize classloader policy, by implementing a small interface (rather than having to subclass the `Object*Stream` classes)

* Pluggable object replacement (also without subclassing)

* Pluggable predefined class tables, which can dramatically decrease stream size and serialization time for stream types which frequently use a common set of classes

* Pluggable predefined instance tables, which make it easy to handle remote references

* Pluggable externalizers which may be used to serialize classes which are not `Serializable` , or for which an alternate strategy is needed

* Customizable stream headers

* Each marshaller instance is highly configurable and tunable to maximize performance based on expected usage patterns

* A generalized API which can support many different protocol implementations, including protocols which do not necessarily provide all the above features

* Inexpensive instance creation, beneficial to applications where many short-lived streams are used

* Support for separate class and instance caches, if the protocol permits; useful for sending multiple messages or requests with a single stream, with separate object graphs but retaining the class cache The 1.2.0 release features many more optimizations to the River protocol implementation, enhanced error reporting (the object stack is included with the exception stack trace), run-time detection of implementations via the [ServiceLoader](http://java.sun.com/javase/6/docs/api/index.html?java/util/ServiceLoader.html "") mechanism, and helper classes to simplify integration with NIO-based applications.