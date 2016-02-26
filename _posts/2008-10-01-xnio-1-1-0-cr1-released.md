---
layout: post
title: XNIO 1.1.0.CR1 Released
categories: 
date: 2008-10-01 16:56:00
redirect_from:
  - /2008/10/xnio-110cr1-released.html
---
 As the title states \- 1.1.0.CR1 is available <a href="http://www.jboss.org/xnio/downloads/">at the XNIO project download page</a>. The main list of changes:

* Added API support for creating one\-way and two\-way pipes.

* Improved separation of the NIO implementation by moving packages and moving the factory classes to a `XnioNio` class.

* Add a `java.util.concurrent.Future` wrapper for `IoFuture` objects.

* Fix a bug with `IoFuture.get()` so that it throws a more specific `CancellationException` when an operation is cancelled.

* Replace SPI module with a more logical abstraction for provider implementations, including some abstract implementations.

* Make `IoFuture.await()` parameter order consistent with other standard timed wait methods.

* Servers changed to bind to `0.0.0.0` by default.

* Added some handy util classes for dealing with `Closeable` resources.

* Consolidated the `IoFuture.Status` value `TIMED_OUT` into `WAITING`, since the two are synonymous anyway, which will simplify `IoFuture.Notifier` implementation among other things.

* Added an abstract base class to allow implementation of one\-line `IoFuture.Notifier` implementations.

