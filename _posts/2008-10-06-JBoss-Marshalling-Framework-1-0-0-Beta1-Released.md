---
layout: post
title: JBoss Marshalling Framework 1.0.0.Beta1 Released
categories: 
date: 2008-10-06 18:46:00
---
 As a part of the JBoss Remoting project (and hopefully several other projects as well), I've developed a separated marshalling (a.k.a. serialization) framework. This framework was inspired by the need for certain features unavailable with the standard `Object*Stream` classes:

* Pluggable class resolvers, making it easy to customize classloader policy, by implementing a small interface (rather than having to subclass the `Object*Stream` classes)

* Pluggable object replacement (also without subclassing)

* Pluggable predefined class tables, which can dramatically decrease stream size and serialization time for stream types which frequently use a common set of classes

* Pluggable predefined instance tables, which make it easy to handle remote references

* Pluggable externalizers which may be used to serialize classes which are not `Serializable` , or for which an alternate strategy is needed

* Customizable stream headers

* Each marshaller instance is highly configurable and tunable to maximize performance based on expected usage patterns

* A generalized API which can support many different protocol implementations, including protocols which do not necessarily provide all the above features

* Inexpensive instance creation, beneficial to applications where many short-lived streams are used

* Support for separate class and instance caches, if the protocol permits; useful for sending multiple messages or requests with a single stream, with separate object graphs but retaining the class cache  
The default implementation, known as "River", is also available as a separate JAR.

This project does not have a home page, but the binaries have been uploaded to [repositry.jboss.org](http://repository.jboss.org/jboss/marshalling/1.0.0.Beta1/lib/ "") and the Javadocs are available online at [docs.jboss.org](http://docs.jboss.org/river/1.0.0.Beta1/api/ "") . The source code may be found in [my sandbox repository](http://anonsvn.jboss.org/repos/sandbox/david.lloyd/jboss-marshalling/trunk/ "") for now, until it gets a permanent home.

Enjoy!