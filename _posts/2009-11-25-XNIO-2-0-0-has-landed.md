---
layout: post
title: XNIO 2.0.0 has landed
categories: 
date: 2009-11-25 16:58:00
---
 XNIO is a simplified low-level I/O layer which can be used anywhere you are using NIO today. It frees you from the hassle of dealing with Selectors and NIO's poor support for SSL, multicast sockets and non-socket I/O, while still maintaining all the capabilities present in NIO. The XNIO 2.0.0 release includes all of the features of the 1.x series, including:

* API compatibility with NIO channels and APIs which consume them

* Powerful callback-based API for channel status changes

* Very simple API for data transfer on channels

* Enhanced NIO buffer support, with many convenience methods to make traditionally difficult buffer manipulation tasks easier

* TCP and UDP client and server support

* API support for other socket types (such as UNIX domain sockets)

* The ability to intermix blocking and non-blocking I/O operations freely and easily

* JMX management for all channels

* Powerful [`IoFuture`](http://docs.jboss.org/xnio/2.0/api/index.html?org/jboss/xnio/IoFuture.html "") interface and support classes simplify asynchronous I/O operation support in XNIO as well as in your application

And these new features:

* SSL channel types for easy SSL support - vastly simpler than the NIO-targeted [`SSLEngine`](http://java.sun.com/javase/6/docs/api/index.html?javax/net/ssl/SSLEngine.html "") API

* New channel listener interface which makes implementing clients and servers even simpler

* Runtime-switchable event listener registration for easy support of "state pattern"-based protocol implementations

* Support for JMX-managed old-I/O [`SocketFactory`](http://java.sun.com/javase/6/docs/api/index.html?javax/net/SocketFactory.html "") and [`ServerSocketFactory`](http://java.sun.com/javase/6/docs/api/index.html?javax/net/ServerSocketFactory.html "") instances to retrofit legacy applications with management capabilities

* Service location API which frees users from a compile-time dependency on an implementation JAR

* A new [User Guide](http://www.jboss.org/xnio/docs "")

* Simplified channel API makes custom channel implementation easy

* Improved generic configuration API via immutable [`OptionMap`](http://docs.jboss.org/xnio/2.0/api/index.html?org/jboss/xnio/OptionMap.html "") class

* Improved API to allow user applications to easily provide [`IoFuture`](http://docs.jboss.org/xnio/2.0/api/index.html?org/jboss/xnio/IoFuture.html "") implementations

* Improved zero-copy integration with NIO's [`FileChannel`](http://java.sun.com/javase/6/docs/api/index.html?java/nio/channels/FileChannel.html "")

* And many more... The project page is at [http://www.jboss.org/xnio](http://www.jboss.org/xnio "") . Download the release from the [download page](http://www.jboss.org/xnio/downloads "") , or in the [JBoss Maven repository](http://repository.jboss.org/maven2 "") under the [`org.jboss.xnio`](http://repository.jboss.org/maven2/org/jboss/xnio "") group ID. The documentation, including Javadoc and the user manual, are available on [the docs page](http://www.jboss.org/xnio/docs "") . Issues can be filed in the project's [JIRA bug tracker](https://jira.jboss.org/jira/browse/XNIO "") .