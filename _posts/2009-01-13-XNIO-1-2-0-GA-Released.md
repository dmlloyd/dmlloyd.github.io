---
layout: post
title: XNIO 1.2.0.GA Released
categories: 
date: 2009-01-13 18:09:00
---
 The final GA release of XNIO 1.2.0 is [available for download](http://www.jboss.org/xnio/downloads "") . This release represents a "settling in" to a 16-week release cycle that should continue from now on. Here's a complete changelog since 1.1.0.GA:

##   Bugs

* [ [XNIO-56](https://jira.jboss.org/jira/browse/XNIO-56 "") ] - Xnio.create() fails for the default provider

* [ [XNIO-57](https://jira.jboss.org/jira/browse/XNIO-57 "") ] - NioSocketChannelImpl suspendWrites method suspends reads instead of writes

* [ [XNIO-58](https://jira.jboss.org/jira/browse/XNIO-58 "") ] - NPE bug in AllocatedMessageChannelStreamChannelHandler

* [ [XNIO-60](https://jira.jboss.org/jira/browse/XNIO-60 "") ] - NIO: When pooled selectors are exhausted, an exception is thrown

* [ [XNIO-65](https://jira.jboss.org/jira/browse/XNIO-65 "") ] - NioPipeTestCase.testOneWayPipeSinkClose fails under Java 5

* [ [XNIO-67](https://jira.jboss.org/jira/browse/XNIO-67 "") ] - TCP tests fail under Java 5, not Java 6

##   Feature Request

* [ [XNIO-19](https://jira.jboss.org/jira/browse/XNIO-19 "") ] - Add ability to switch a channel between blocking and non-blocking

* [ [XNIO-50](https://jira.jboss.org/jira/browse/XNIO-50 "") ] - Mitigate the performance impact of Selector.wakeup()

* [ [XNIO-54](https://jira.jboss.org/jira/browse/XNIO-54 "") ] - API for one-time servers

* [ [XNIO-64](https://jira.jboss.org/jira/browse/XNIO-64 "") ] - IoFuture notifiers with attachments ( **see below** )

* [ [XNIO-66](https://jira.jboss.org/jira/browse/XNIO-66 "") ] - Add static blocking read/write methods to Channels

##   Task

* [ [XNIO-17](https://jira.jboss.org/jira/browse/XNIO-17 "") ] - Add monitoring via JMX

* [ [XNIO-39](https://jira.jboss.org/jira/browse/XNIO-39 "") ] - Remove deprecated Xnio.createNio() methods

* [ [XNIO-48](https://jira.jboss.org/jira/browse/XNIO-48 "") ] - Create a UNIX domain socket API

* [ [XNIO-64](https://jira.jboss.org/jira/browse/XNIO-64 "") ] - IoFuture notifiers with attachments

##   Links

Download: [http://www.jboss.org/xnio/downloads](http://www.jboss.org/xnio/downloads "")

Javadoc: [http://docs.jboss.org/xnio/1.2.0.GA/api](http://docs.jboss.org/xnio/1.2.0.GA/api "")

Project page: [http://www.jboss.org/xnio](http://www.jboss.org/xnio "")

Bug tracker: [https://jira.jboss.org/jira/browse/XNIO](https://jira.jboss.org/jira/browse/XNIO "")

Just a reminder that the 1.2.0.GA release is **not 100% binary- and source-compatible with 1.1.x** , so if you choose to move to 1.2.0.GA, make sure you do a clean build of your project to identify any potential problems with the changes surrounding the new `IoFuture` attachment feature.