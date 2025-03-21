---
layout: post
title: Safely downgrading a write lock with ReadWriteLock
description: A simple pattern to safely downgrade a write lock to a read lock safely.
categories: 
date: 2010-02-23 14:45:00
author: dmlloyd
aliases:
  - /safely-downgrading-a-write-lock-with-readwritelock/
  - /2010/02/safely-downgrading-write-lock-with.html
---
 A simple pattern to downgrade a write lock to a read lock safely:

```java
ReadWriteLock rwl = getLock();
Lock lock = rwl.writeLock();
lock.lock();
try {
    // <<... Perform write operation ...>>
    // downgrade lock safely
    final Lock readLock = rwl.readLock();
    readLock.lock(); // Possible failure #1
    try {
       lock.unlock(); // Possible failure #2
    } finally {
        lock = readLock;
    }
} finally {
    lock.unlock();
}
```

The reason this works is as follows: 1. Anything which fails before possible failure #1 will cause the original lock to unlock, due to the `finally` block. 2. If possible failure #1 occurs, the read lock is never locked, thus the write lock is still unlocked in the outer `finally` block. 3. If possible failure #2 occurs, the now\-locked read lock is stashed into `lock` thanks to the inner `finally` block, thus the read lock is released in the outer `finally` block. At this point, no matter what, the write lock is released. 4. Any failure after possible failure #2 will cause the outer `finally` block to release the read lock. The nice thing about this approach is that you can even downgrade the lock inside of a conditional and the structure is not broken.
