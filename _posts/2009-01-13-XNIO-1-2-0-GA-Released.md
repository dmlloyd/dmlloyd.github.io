---
layout: post
title: XNIO 1.2.0.GA Released
categories: 
date: 2009-01-13 18:09:00
---
 The final GA release of XNIO 1.2.0 is <a href="http://www.jboss.org/xnio/downloads">available for download</a>. This release represents a "settling in" to a 16\-week release cycle that should continue from now on. Here's a complete changelog since 1.1.0.GA:

##   Bugs

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-56">XNIO-56</a>\] \- Xnio.create() fails for the default provider

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-57">XNIO-57</a>\] \- NioSocketChannelImpl suspendWrites method suspends reads instead of writes

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-58">XNIO-58</a>\] \- NPE bug in AllocatedMessageChannelStreamChannelHandler

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-60">XNIO-60</a>\] \- NIO: When pooled selectors are exhausted, an exception is thrown

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-65">XNIO-65</a>\] \- NioPipeTestCase.testOneWayPipeSinkClose fails under Java 5

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-67">XNIO-67</a>\] \- TCP tests fail under Java 5, not Java 6

##   Feature Request

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-19">XNIO-19</a>\] \- Add ability to switch a channel between blocking and non\-blocking

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-50">XNIO-50</a>\] \- Mitigate the performance impact of Selector.wakeup()

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-54">XNIO-54</a>\] \- API for one\-time servers

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-64">XNIO-64</a>\] \- IoFuture notifiers with attachments ( **see below**)

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-66">XNIO-66</a>\] \- Add static blocking read/write methods to Channels

##   Task

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-17">XNIO-17</a>\] \- Add monitoring via JMX

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-39">XNIO-39</a>\] \- Remove deprecated Xnio.createNio() methods

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-48">XNIO-48</a>\] \- Create a UNIX domain socket API

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-64">XNIO-64</a>\] \- IoFuture notifiers with attachments

##   Links

Download: <a href="http://www.jboss.org/xnio/downloads">http://www.jboss.org/xnio/downloads</a>

Javadoc: <a href="http://docs.jboss.org/xnio/1.2.0.GA/api">http://docs.jboss.org/xnio/1.2.0.GA/api</a>

Project page: <a href="http://www.jboss.org/xnio">http://www.jboss.org/xnio</a>

Bug tracker: <a href="https://jira.jboss.org/jira/browse/XNIO">https://jira.jboss.org/jira/browse/XNIO</a>

Just a reminder that the 1.2.0.GA release is **not 100% binary\- and source\-compatible with 1.1.x**, so if you choose to move to 1.2.0.GA, make sure you do a clean build of your project to identify any potential problems with the changes surrounding the new `IoFuture` attachment feature.