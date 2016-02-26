---
layout: post
title: XNIO 2.0.2 Bugfix Release
categories: 
date: 2010-01-25 14:21:00
---
 A new release of [XNIO]("http://jboss.org/xnio" "") is available which fixes several bugs. For more information about XNIO 2.0, see [this previous post]("http://dmlloyd.blogspot.com/2009/11/xnio-200-has-landed.html" "") . Users of XNIO 2.0.0 and 2.0.1 are encouraged to update to 2.0.2 to take advantage of these bug fixes:

* [ [XNIO-82]("https://jira.jboss.org/jira/browse/XNIO-82" "") ] - Remove jboss-classloading.xml from artifact

* [ [XNIO-83]("https://jira.jboss.org/jira/browse/XNIO-83" "") ] - Getting an option from a string fails due to typo

* [ [XNIO-84]("https://jira.jboss.org/jira/browse/XNIO-84" "") ] - Enable all supported SSL cipher suites if none are explicitly enabled

* [ [XNIO-85]("https://jira.jboss.org/jira/browse/XNIO-85" "") ] - Possible deadlock in SSL negotiation phase

* [ [XNIO-86]("https://jira.jboss.org/jira/browse/XNIO-86" "") ] - CCE produced when casting a null option value

* [ [XNIO-87]("https://jira.jboss.org/jira/browse/XNIO-87" "") ] - XNIO log messages don't preserve the source class/method name

* [ [XNIO-88]("https://jira.jboss.org/jira/browse/XNIO-88" "") ] - Open listener specification is now optional; do not throw NPE if it is not provided The release is available from the [downloads page]("http://jboss.org/xnio/downloads" "") .