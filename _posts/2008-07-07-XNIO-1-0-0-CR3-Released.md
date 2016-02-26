---
layout: post
title: XNIO 1.0.0.CR3 Released
categories: 
date: 2008-07-07 18:50:00
---
 Well, I lied. I've found a couple more minor API issues which I've fixed for CR3 - a misspelling in a method name ("interruptible", not "interruptable"), a typo which prevented junit from being downloaded, one interface rename ("Client" becomes "ChannelSource"), and a few other minor fixes are included in this release. I think this one will become GA unless I find a glaring bug in the next week or so.  
Once again, the release is available at the [downloads page](http://www.jboss.org/xnio/downloads/ "") of the [XNIO project website](http://www.jboss.org/xnio/ "") .  
Release Notes - XNIO - Version 1.0.0.CR3

##   Bug

* [ [XNIO-27](http://jira.jboss.com/jira/browse/XNIO-27 "") ] - "Interruptibly" is misspelled

* [ [XNIO-28](http://jira.jboss.com/jira/browse/XNIO-28 "") ] - Streams classes do not belong in XNIO

* [ [XNIO-30](http://jira.jboss.com/jira/browse/XNIO-30 "") ] - Build fails due to incorrect license name for junit

* [ [XNIO-31](http://jira.jboss.com/jira/browse/XNIO-31 "") ] - Logging methods might throw exceptions

##  Task

* [ [XNIO-14](http://jira.jboss.com/jira/browse/XNIO-14 "") ] - Use a service locator pattern to locate XNIO providers in the standalone API

* [ [XNIO-29](http://jira.jboss.com/jira/browse/XNIO-29 "") ] - Rename org.jboss.xnio.Client to "ChannelSource"