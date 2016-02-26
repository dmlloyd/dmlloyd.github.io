---
layout: post
title: XNIO 1.0.0.CR1 Released
categories: 
date: 2008-06-11 11:10:00
redirect_from:
  - /2008/06/xnio-100cr1-released.html
---
 I've released XNIO 1.0.0.CR1. XNIO is a replacement for NIO that vastly simplifies the handling of non\-blocking I/O. XNIO gives you the full power of NIO channels while eliminating the headache of thread and Selector management. Also, it is dramatically simpler to implement an XNIO provider than it is to implement an NIO provider \- opening the door to APR or other native implementations as well as non\-network transport types such as serial ports, and platform\-specific types like UNIX domain sockets.

The release is available from the <a href="http://www.jboss.org/xnio/">XNIO project page</a>.