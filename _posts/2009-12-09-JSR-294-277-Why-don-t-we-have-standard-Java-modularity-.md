---
layout: post
title: JSR 294/277: Why don't we have standard Java modularity?
categories: 
date: 2009-12-09 21:33:00
---
 Why is it so difficult to write a modularity specification for Java? The requirements appear to be relatively simple:

0. Provide an API whereby a user can load a module and link to it (in other words, get a classloader which can be used to load exported classes), possibly with a mechanism to "run" a module, similar to how one can execute "java -jar foo.jar" today

0. Support a per-module access control level, in the Java language and JVM

0. Provide a way to specify what other modules are required/optional, and whether they are imported or imported and exported (in other words, "metadata")

0. Provide a way to recursively locate, load, register, and link referenced modules in an efficient manner (in other words, "resolution") So what would it take to implement this? Not much from what I can see.  
The user API would amount to a new ModuleLoader class which has API elements to load a module and return an instance of a new Module class, which has the ability to either access the exported classes therein directly, or get at a ClassLoader which can do so.  
A per-module access control level can be implemented by one simple rule: If a class is loaded from a module's ClassLoader, change "package-private" to mean "module-private" instead. Otherwise, stick to the old rules. Very straightforward.  
Metadata can be implemented very simply as plain data objects which are read by the appropriate ModuleLoader, or even as an implementation detail of ModuleLoader itself.  
Resolution could (and should) be a pluggable thing within a ModuleLoader. This would enable customized handling like the ability to load modules right out of Maven, or the ability to download modules on demand from a trusted remote repository.  
So what's the deal? Wouldn't this small group of classes (and one small JLS/JVM change) be sufficient to solve the problem?