---
layout: post
title: Binding to privileged ports with XNIO
categories: 
date: 2008-10-23 09:58:00
---
A couple random ideas about how to bind privileged (<1024) port numbers using XNIO. It might be possible to utilize the UNIX domain socket API (slated for 1.2.0) and use the file descriptor-passing mechanism. A root-privileged server (written in C) could be started that binds to a UNIX domain socket on the filesystem, whose access permissions are limited. The XNIO-based application can use this domain socket to request bound ports and get the file descriptor back, which it would use to construct a server. Another idea would be to turn it around, and have the XNIO process run a setuid root program (via ProcessBuilder) which binds the port and hands it back through a socket created by the Java program.