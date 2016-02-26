---
layout: post
title: XNIO 1.2.0.CR2 Released
categories: 
date: 2009-01-07 20:47:00
---
 The XNIO 1.2.0.CR2 release is up on [the project page]("http://www.jboss.org/xnio" "") . This release features cleaned up Javadoc, several bug fixes, and a few handy new features (in particular, static methods to perform blocking reads/writes for most common channel types).

The biggest change is the ability to pass an attachment in to an `IoFuture` instance, via the [`IoFuture.addNotifier()`]("http://docs.jboss.org/xnio/1.2.0.CR2/api/org/jboss/xnio/IoFuture.html#addNotifier(org.jboss.xnio.IoFuture.Notifier,%20A)" "") method. This allows users to reuse notifier instances, which is a very powerful capability. Unfortunately there was no clean way to add this feature without breaking compatibility in some way, so I went for the cleanest implementation possible, which means that **XNIO 1.2.x is not 100% binary- and source-compatible with XNIO 1.1.x** . I hope that this feature is useful enough that you will all forgive me for that change. Because of this change I will maintain the 1.1.x series for longer than I otherwise might, in case there are users out there who are committed to the old API.

Read the Javadoc [here]("http://docs.jboss.org/xnio/1.2.0.CR2/api" "") , download the release [here]("http://www.jboss.org/xnio/downloads/" "") , report bugs [here]("http://jira.jboss.org/jira/browse/XNIO" "") .