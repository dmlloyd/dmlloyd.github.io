---
layout: post
title: XNIO 2.0.2 Bugfix Release
categories: 
date: 2010-01-25 14:21:00
---
 A new release of <a href="http://jboss.org/xnio">XNIO</a>is available which fixes several bugs. For more information about XNIO 2.0, see <a href="http://dmlloyd.blogspot.com/2009/11/xnio-200-has-landed.html">this previous post</a>. Users of XNIO 2.0.0 and 2.0.1 are encouraged to update to 2.0.2 to take advantage of these bug fixes:

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-82">XNIO-82</a>\] \- Remove jboss\-classloading.xml from artifact

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-83">XNIO-83</a>\] \- Getting an option from a string fails due to typo

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-84">XNIO-84</a>\] \- Enable all supported SSL cipher suites if none are explicitly enabled

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-85">XNIO-85</a>\] \- Possible deadlock in SSL negotiation phase

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-86">XNIO-86</a>\] \- CCE produced when casting a null option value

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-87">XNIO-87</a>\] \- XNIO log messages don't preserve the source class/method name

* \[ <a href="https://jira.jboss.org/jira/browse/XNIO-88">XNIO-88</a>\] \- Open listener specification is now optional; do not throw NPE if it is not provided The release is available from the <a href="http://jboss.org/xnio/downloads">downloads page</a>.