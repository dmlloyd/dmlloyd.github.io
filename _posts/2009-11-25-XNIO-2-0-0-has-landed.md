---
layout: post
title: XNIO 2.0.0 has landed
categories: 
date: 2009-11-25 16:58:00
---
 XNIO is a simplified low\-level I/O layer which can be used anywhere you are using NIO today. It frees you from the hassle of dealing with Selectors and NIO's poor support for SSL, multicast sockets and non\-socket I/O, while still maintaining all the capabilities present in NIO. The XNIO 2.0.0 release includes all of the features of the 1.x series, including:

* API compatibility with NIO channels and APIs which consume them

* Powerful callback\-based API for channel status changes

* Very simple API for data transfer on channels

* Enhanced NIO buffer support, with many convenience methods to make traditionally difficult buffer manipulation tasks easier

* TCP and UDP client and server support

* API support for other socket types (such as UNIX domain sockets)

* The ability to intermix blocking and non\-blocking I/O operations freely and easily

* JMX management for all channels

* Powerful <a href="http://docs.jboss.org/xnio/2.0/api/index.html?org/jboss/xnio/IoFuture.html"><code>IoFuture</code></a>interface and support classes simplify asynchronous I/O operation support in XNIO as well as in your application

And these new features:

* SSL channel types for easy SSL support \- vastly simpler than the NIO\-targeted <a href="http://java.sun.com/javase/6/docs/api/index.html?javax/net/ssl/SSLEngine.html"><code>SSLEngine</code></a>API

* New channel listener interface which makes implementing clients and servers even simpler

* Runtime\-switchable event listener registration for easy support of "state pattern"\-based protocol implementations

* Support for JMX\-managed old\-I/O <a href="http://java.sun.com/javase/6/docs/api/index.html?javax/net/SocketFactory.html"><code>SocketFactory</code></a>and <a href="http://java.sun.com/javase/6/docs/api/index.html?javax/net/ServerSocketFactory.html"><code>ServerSocketFactory</code></a>instances to retrofit legacy applications with management capabilities

* Service location API which frees users from a compile\-time dependency on an implementation JAR

* A new <a href="http://www.jboss.org/xnio/docs">User Guide</a>

* Simplified channel API makes custom channel implementation easy

* Improved generic configuration API via immutable <a href="http://docs.jboss.org/xnio/2.0/api/index.html?org/jboss/xnio/OptionMap.html"><code>OptionMap</code></a>class

* Improved API to allow user applications to easily provide <a href="http://docs.jboss.org/xnio/2.0/api/index.html?org/jboss/xnio/IoFuture.html"><code>IoFuture</code></a>implementations

* Improved zero\-copy integration with NIO's <a href="http://java.sun.com/javase/6/docs/api/index.html?java/nio/channels/FileChannel.html"><code>FileChannel</code></a>

* And many more... The project page is at <a href="http://www.jboss.org/xnio">http://www.jboss.org/xnio</a>. Download the release from the <a href="http://www.jboss.org/xnio/downloads">download page</a>, or in the <a href="http://repository.jboss.org/maven2">JBoss Maven repository</a>under the <a href="http://repository.jboss.org/maven2/org/jboss/xnio"><code>org.jboss.xnio</code></a>group ID. The documentation, including Javadoc and the user manual, are available on <a href="http://www.jboss.org/xnio/docs">the docs page</a>. Issues can be filed in the project's <a href="https://jira.jboss.org/jira/browse/XNIO">JIRA bug tracker</a>.