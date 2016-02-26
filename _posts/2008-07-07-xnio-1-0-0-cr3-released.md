---
layout: post
title: XNIO 1.0.0.CR3 Released
categories: 
date: 2008-07-07 18:50:00
redirect_from:
  - /2008/07/xnio-100cr3-released.html
---
 Well, I lied. I've found a couple more minor API issues which I've fixed for CR3 \- a misspelling in a method name ("interruptible", not "interruptable"), a typo which prevented junit from being downloaded, one interface rename ("Client" becomes "ChannelSource"), and a few other minor fixes are included in this release. I think this one will become GA unless I find a glaring bug in the next week or so.

Once again, the release is available at the <a href="http://www.jboss.org/xnio/downloads/">downloads page</a>of the <a href="http://www.jboss.org/xnio/">XNIO project website</a>.

Release Notes \- XNIO \- Version 1.0.0.CR3

##   Bug

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-27">XNIO-27</a>\] \- "Interruptibly" is misspelled

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-28">XNIO-28</a>\] \- Streams classes do not belong in XNIO

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-30">XNIO-30</a>\] \- Build fails due to incorrect license name for junit

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-31">XNIO-31</a>\] \- Logging methods might throw exceptions

##  Task

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-14">XNIO-14</a>\] \- Use a service locator pattern to locate XNIO providers in the standalone API

* \[ <a href="http://jira.jboss.com/jira/browse/XNIO-29">XNIO-29</a>\] \- Rename org.jboss.xnio.Client to "ChannelSource"