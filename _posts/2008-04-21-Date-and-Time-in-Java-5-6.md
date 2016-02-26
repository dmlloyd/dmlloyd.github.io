---
layout: post
title: Date and Time in Java 5 & 6
categories: 
date: 2008-04-21 10:44:00
---
 For those of you who aren't aware, [JSR-310](https://jsr-310.dev.java.net/ "") is devoted to developing a (much-needed) new and improved replacement date and time API for Java. Since the spec lead, Stephen Colebourne (who is the author of the excellent [JODA date and time library](http://joda-time.sourceforge.net/index.html "") ), has made the wise decision to make the process public for the JSR, you can read (and join) the discussions taking place around this effort.  
One [discussion I'd like to highlight](https://jsr-310.dev.java.net/servlets/BrowseList?listName=dev&by=thread&from=1117076&to=1117076&first=1&count=6 "") is a bit of talk over the last few days regarding the final package in which the date and time API will finally reside. Currently the API is set to end up in the "javax.date" package. This should allow us (from a legal perspective) to be able to provide the date and time implementation as a standalone library. Great news for folks who, for whatever reason, are not going to be able to upgrade to Java 7 (which is the target for this JSR) the day it comes out. This would allow that (sizeable) population to use the new API in their existing systems.  
Also, have a look at the [latest API javadoc](https://jsr-310.dev.java.net/nonav/doc-2008-04-20/index.html "") for a taste of what the API looks like currently.