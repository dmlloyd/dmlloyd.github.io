---
layout: post
title: AIO versus network servers
categories: 
date: 2008-05-22 14:38:00
---
 For those who aren't aware, the AIO read idiom works something like this pseudocode:

    buffer = allocate_buffer();  
    future = aio_channel.read(buffer); // read takes place in the background; this call does not block  
    future.set_completion_handler(handler);  
    return;

And later on in handler:

    void handler(Buffer buffer) {  
     ... decode the received buffer ...  
    }

Why is this not right for high-connection-count network servers? Imagine your buffer size is 1K. Imagine you've got 10k concurrent connections. You need a buffer to be allocated before you can kick off an async read. That's an initial 10 megabytes in idle buffers, not to mention a *minimum* of 10k buffer allocations - even if the connection is never used or the channel never becomes readable.

This problem is similar to [the problem that Jean-Francois Arcand is referring to](http://weblogs.java.net/blog/jfarcand/archive/2006/06/tricks_and_tips.html "") in his NIO Tips & Tricks blog.

For this reason, I still believe that multiplexing/testing for readiness is still the best approach in terms of scalability. And multiplexing for socket channels obviates the need for AIO: if a channel is readable, then a read() will by definition not block (the data is already in the receive buffer). Therefore AIO is not useful for waiting for a socket to become ready for reading.

Writing is a different story. When you want to write to a socket channel, to minimize latency generally your write data is assembled in a buffer, and then write is invoked. In the worst case, with regular non-blocking IO, the write returns a 0 (in POSIX-ish C it would be an EAGAIN result) indicating that the write would have blocked (that is, the kernel write buffer has no space available, so no data can be sent right now). With non-blocking I/O, what you do at this point is register for writable events, and your handler gets notified when the channel is writable. At this stage you retry your write.

Note that even now, your entire buffer might not be written, especially if you're using a stream socket type such as TCP. The reason is that your buffer might still be larger than the kernel's send buffer. So in this case, you reregister your handler to be notified on writes again, repeating until your entire buffer was sent.

This is a lot of boilerplate for something that's fairly common. In this case, it's far easier to assemble your buffer, kick off an async write, and call it a day. You'll get a notification when your write is complete, without monkeying around with registering and re-registering your handler.