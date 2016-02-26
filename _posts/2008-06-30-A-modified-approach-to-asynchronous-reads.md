---
layout: post
title: A modified approach to asynchronous reads
categories: 
date: 2008-06-30 17:30:00
---
 In a [previous post](http://dmlloyd.blogspot.com/2008/05/aio-versus-network-servers.html "") , I talked about the impracticality of asynchronous reads in a scalable server situation. The problem I cited was that if there are large amounts of pending reads, each with its own preallocated buffer, a large quantity of resources can possibly be consumed, thus impeding scalability.

I've been thinking about the problem some more and I've come up with an alternate approach. Basically, the solution to the problem is to simply not allocate a buffer until the read takes place. This is made pretty simple by use of the [`BufferAllocator` interface](http://docs.jboss.org/xnio/1.0/api/org/jboss/xnio/BufferAllocator.html "") in [XNIO](http://www.jboss.org/xnio/ "") .

Using this interface, the signature of the asynchronous read method would look like this:

`IoFuture asyncRead(BufferAllocator allocator) throws IOException;`

The buffer is allocated only when the channel is readable. And if an NIO.2-style (or similar) async read is used "under the covers" for whatever reason, then the allocation can simply happen right upfront.

Look for this feature in XNIO 1.1!