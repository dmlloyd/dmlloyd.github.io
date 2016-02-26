---
layout: post
title: Class-local data
categories: 
date: 2009-02-25 17:48:00
---


Today I ran across a problem that I've hit frequently in the past, this time while designing an extension to [JBoss Marshalling](http://www.jboss.org/jbossmarshalling "") which allows users to annotate classes which are to be specially externalized. The problem is that one wants to "remember" what `Externalizer` instances go with what classes on a semi-permanent basis; however, one does not wish to leak class references (which can lead to the permanent generation filling up, and your application crashing with an `OutOfMemoryError` , especially if you're frequently loading and unloading classes).

The traditional solution to keep a leak-resistant cache is to employ a `WeakHashMap` like so (usually in conjunction with some concurrency strategy, like a `Lock` ):

      private static final Map<Class , Externalizer> cache = new WeakHashMap<Class , Externalizer>();

The `WeakHashMap` retains a weak reference to the key values (in this case, the `Class` instance), so if no strong references remain, the `Class` is unloaded, and all is well. However, in this particular use case, the `Externalizer` instance very likely will itself hold a strong reference to the `Class` . Why? Because in JBoss Marshalling, `Externalizer` instances may be custom-designed with a single class in mind; thus, it might have a field or a constructor call of that class' type, which means strong reference. Since `WeakHashMap` retains a strong reference to its value, this means that there is a strong reference from my cache to the class after all, so all is for naught.

So I got to thinking - what if I could stash my `Externalizer` instance on the `Class` itself? Then I wouldn't even need to keep a cache at all, and the only reference to the `Class` that I could cause to be would be from the `Class` itself. That would be awesome! Of course, we wouldn't want anyone else to be able to get at my data; it could be private to me, after all. And we wouldn't want it to leak the other way - the `Class` should not hold a direct strong reference to my class, yet we don't exactly want it to be a weak reference either (because then the data might disappear before I've had a chance to use it).

So my idea is to create a new type of storage: class-local data. You'd access it similarly to thread-local data, except instead of being keyed from the current thread, you'd give it a class:

      // The methods are similar to the corresponding methods on Map and ConcurrentMap  
      public final class ClassLocal<T> {  
         public T get(Class  clazz) { ... }  
         public T put(Class  clazz, T value) { ... }  
         public boolean hasData(Class  clazz) { ... }  
         public T putIfAbsent(Class  clazz, T value) { ... }  
         public T remove(Class  clazz) { ... }  
         public boolean remove(Class  clazz, T oldValue) { ... }  
         public T replace(Class  clazz, T newValue) { ... }  
         public boolean replace(Class  clazz, T oldValue, T newValue) { ... }  
      }  
      private final ClassLocal<Externalizer> classLocal = new ClassLocal<Externalizer>();  
      ...in some method...  
      Externalizer externalizer = classLocal.get(SomeClass.class);

A `ClassLocal` instance looks and acts a lot like a map, but with one important difference: the data itself is stored on the `Class` instance, not in the `ClassLocal` . Implementation-wise this could be accomplished via a `WeakHashMap` or similar structure on the `Class` itself (with a concurrency strategy) similar to this:

      public final class Class<T> {  
         ...  
         final Map<ClassLocal , Object> classLocalData = Collections.synchronizedMap(new WeakHashMap<ClassLocal , Object>());  
      }

...which could then be accessed directly by the `ClassLocal` implementation. Real JDK experts could probably think of a few ways to optimize this - e.g. avoid creating a map instance until someone puts data there, come up with a more clever locking strategy, etc.

**Update:** As I ought to have expected, I'm nowhere near the first person ever to let their mind wander down this path - there's a [Sun bug (#6493635)](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6493635 "") about it, as well as articles by such clever folks as [Crazy Bob](http://crazybob.org/2006/12/caching-class-related-information.html "") and [Jacob Hookom](http://weblogs.java.net/blog/jhook/archive/2006/12/class_metadata.html "") about this very topic.