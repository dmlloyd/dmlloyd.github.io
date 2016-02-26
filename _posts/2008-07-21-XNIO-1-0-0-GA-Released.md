---
layout: post
title: XNIO 1.0.0.GA Released
categories: 
date: 2008-07-21 19:29:00
---
 The first GA release of XNIO is now available. There is only one substantial fix in this release. The standalone API did not present a way to close ChannelSource and Connector instances that were created via an Xnio instance. This has been fixed, and I think the wait has been long enough.

Find the release at the <a href="http://www.jboss.org/xnio/downloads/">downloads page</a>of the <a href="http://www.jboss.org/xnio/">XNIO project website</a>. Also check out the <a href="http://docs.jboss.org/xnio/1.0/api/index.html">online documentation</a>.