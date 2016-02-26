---
layout: post
title: More uses for Dave's Dumb Closures
categories: 
date: 2008-10-07 09:42:00
---
 Truly pluggable locking strategies:

        public interface Locker {  
            void hold() for myBlock();  
        }  
        public class SynchLocker {  
            private final Object lock;  
            public SynchLocker(Object lock) { this.lock = lock; }  
            public void hold() for myBlock() {  
                synchronized (lock) {  
                    myBlock();  
                }  
            }  
        }  
        public class LockLocker {  
            private final Lock lock;  
            public LockLocker(Lock lock) { this.lock = lock; }  
            public void hold() for myBlock() {  
                lock.lock();  
                try {  
                    myBlock();  
                } finally {  
                    lock.unlock();  
                }  
            }  
        }  
        ...later...  
        Locker l = ...take your pick...;  
        l.hold() {  
            ...do stuff...  
        }

Let's see you do that efficiently with anonymous inner classes or BGGA.