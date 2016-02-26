---
layout: post
title: Minor XNIO re-org for 1.2.0
categories: 
date: 2008-10-16 11:17:00
---
 For XNIO 1.2.0, I'm doing away with the separated API versus standalone JAR. My original notion was that the standalone JAR would contain the whole API, plus the `Xnio` bootstrapping class, plus at least one implementation (NIO right now, as that is currently the only complete implementation). But I think it will be simpler going forward to just add `Xnio` into the API itself, and ship an API JAR plus one JAR per implementation. So from now on (including any future snapshots), the JARs will follow this new structure instead.