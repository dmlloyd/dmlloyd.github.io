---
layout: post
title: XNIO 1.0.0.CR2 Released
categories: 
date: 2008-06-23 18:55:00
redirect_from:
  - /2008/06/xnio-100cr2-released.html
---


This new version fixes several problems in CR1 and is closer to a final product now. I do not anticipate any further API changes, so this version is safe to develop against in anticipation for the final GA release (which I have marked for July 28 \- not too far in the future, but hopefully far enough that any remaining bugs will be found and flushed out).

For those new to XNIO, it is a replacement for NIO that vastly simplifies the handling of non\-blocking I/O. To quote my tag line from last time: XNIO gives you the full power of NIO channels while eliminating the headache of thread and Selector management.

The release is available at the <a href="http://www.jboss.org/xnio/downloads/">downloads page of the XNIO project</a>. Release Notes \- XNIO \- Version 1.0.0.CR2

##  Bug

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-20">XNIO-20</a>\] \- Channel options support is incomplete

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-22">XNIO-22</a>\] \- No way to shut down server connections

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-23">XNIO-23</a>\] \- An exception in handleOpened should always close the connection

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-25">XNIO-25</a>\] \- Standalone API isn't locking properly

##  Feature Request

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-21">XNIO-21</a>\] \- Samples directory with a few simple examples

##  Task

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-18">XNIO-18</a>\] \- License files and file headers

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-26">XNIO-26</a>\] \- Options should be changed to typesafe constants