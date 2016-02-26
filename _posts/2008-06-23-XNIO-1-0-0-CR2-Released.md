---
layout: post
title: XNIO 1.0.0.CR2 Released
categories: 
date: 2008-06-23 18:55:00
---


This new version fixes several problems in CR1 and is closer to a final product now. I do not anticipate any further API changes, so this version is safe to develop against in anticipation for the final GA release (which I have marked for July 28 - not too far in the future, but hopefully far enough that any remaining bugs will be found and flushed out).

For those new to XNIO, it is a replacement for NIO that vastly simplifies the handling of non-blocking I/O. To quote my tag line from last time: XNIO gives you the full power of NIO channels while eliminating the headache of thread and Selector management.

The release is available at the [downloads page of the XNIO project]("http://www.jboss.org/xnio/downloads/" "") . Release Notes - XNIO - Version 1.0.0.CR2

##  Bug

* [ [XNIO-20]("http://jira.jboss.com/jira/browse/XNIO-20" "") ] - Channel options support is incomplete

* [ [XNIO-22]("http://jira.jboss.com/jira/browse/XNIO-22" "") ] - No way to shut down server connections

* [ [XNIO-23]("http://jira.jboss.com/jira/browse/XNIO-23" "") ] - An exception in handleOpened should always close the connection

* [ [XNIO-25]("http://jira.jboss.com/jira/browse/XNIO-25" "") ] - Standalone API isn't locking properly

##  Feature Request

* [ [XNIO-21]("http://jira.jboss.com/jira/browse/XNIO-21" "") ] - Samples directory with a few simple examples

##  Task

* [ [XNIO-18]("http://jira.jboss.com/jira/browse/XNIO-18" "") ] - License files and file headers

* [ [XNIO-26]("http://jira.jboss.com/jira/browse/XNIO-26" "") ] - Options should be changed to typesafe constants