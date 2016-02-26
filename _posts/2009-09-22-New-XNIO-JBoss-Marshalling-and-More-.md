---
layout: post
title: New XNIO, JBoss Marshalling, and More...
categories: 
date: 2009-09-22 10:44:00
---
 Since my last post several months ago, I've been quite busy with a number of project releases.

* JBoss Marshalling version 1.2.0.CR3 - download [here]("http://jboss.org/jbossmarshalling/downloads/" "") , JIRA [here]("https://jira.jboss.org/jira/browse/JBMAR" "") . Includes a much more efficient River protocol implementation, and the Java-compatible Serial implementation, both of which have been measured to outperform Java serialization by 50% or more in some cases, while retaining compliance with the Serialization specification.

* XNIO 2.0.0.CR2 - download [here]("http://jboss.org/xnio/downloads/" "") , JIRA [here]("https://jira.jboss.org/jira/browse/XNIO" "") . Includes an improved API, as well as SSL support, which I think will be quite appreciated by anyone who has ever tried to use [SSLEngine]("http://java.sun.com/javase/6/docs/api/index.html?javax/net/ssl/SSLEngine.html" "") with NIO.

* JBoss LogManager 1.1.0.GA - available from the [JBoss Maven Repository]("http://repository.jboss.org/maven2/org/jboss/logmanager/jboss-logmanager/1.1.0.GA/" "") . A complete re-implementation of the bug-ridden JDK [LogManager implementation]("http://java.sun.com/javase/6/docs/api/index.html?java/util/logging/LogManager.html" "") , with additional handler types (including log4j compatibility), designed specifically to provide a highly efficient and cleanly implemented logging layer for JBossAS.

* JBoss Logging 2.2.0.CR1 - available from the [JBoss Maven Repository]("http://repository.jboss.org/maven2/org/jboss/logging/" "") . An improved version of the logging facade historically used by JBoss projects which run within JBossAS. This new version improves memory usage by providing the ability for backend implementations to cache and reuse instances in a way that is not subject to the classical ClassLoader issues of other similar frameworks. Also provides logging methods which take advantage of the JDK5 [java.util.Formatter]("http://java.sun.com/javase/6/docs/api/index.html?java/util/Formatter.html" "") class. There are a number of other projects in the works as well, so stay tuned!