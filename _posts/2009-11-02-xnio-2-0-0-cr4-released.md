---
layout: post
title: XNIO 2.0.0.CR4 Released
categories: 
date: 2009-11-02 14:30:00
redirect_from:
  - /2009/11/xnio-200cr4-released.html
---
 XNIO 2.0 is getting very close to completion. The latest CR, 2.0.0.CR4, is now available from <a href="http://www.jboss.org/xnio/downloads">the downloads page</a>.

The main change since CR3 is the introduction of a new "Result" interface which acts as feed\-in to an IoFuture. This makes it easy for developers to create an API which exposes IoFuture results to the user, without worrying as much about implementation details. Various support classes have also been added for this purpose. In addition, the implementation classes have been ported to use the new API.