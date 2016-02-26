---
layout: post
title: XNIO 1.1.1.GA Released
categories: 
date: 2008-11-12 18:39:00
---
 The release available on [the downloads page](http://www.jboss.org/xnio/downloads/ "") . This is a bugfix release; I encourage all users of 1.1.0.GA to upgrade.

Here's the list of changes:

* Fix a problem in the `Xnio.create()` method's default provider lookup

* Fix a problem where thread interruption could cause a tight spinloop

* Fix a minor problem in the logger init code

* Merge the improved test log formatter from trunk

Enjoy!