---
layout: post
title: A lightweight readers/writer lock
categories: 
date: 2017-10-30 12:00:00
---

In our <a href="https://github.com/jboss-msc/jboss-msc">JBoss MSC</a> project, it has become necessary to introduce a multiple-readers/single-writer style of lock, however the JDK's ReadWriteLock APIs are too heavy for the job.  Our solution is to implement such a thing using only an ```int``` field and synchronization.

### Theory of operation

The basic idea behind a readers/writer lock is that read locks are shared, meaning multiple threads can proceed holding read locks at the same time, whereas write locks are exclusive, meaning no other thread may hold any other kind of lock as long as the write lock is held.

In this particular implementation, we use an ```int``` field to track the number of active readers.  The synchronization itself is used for the write lock, using the monitor's built-in condition ```wait```/```notify``` to block and unblock waiting writers.  When a reader lock is held, the monitor is not held but the reader count is greater than zero.

### Implementation

Here's the basic framework of the lock code, shown here as an abstract base class:

```java
public abstract class ReadWriteLockable {
    private int readerCount;

    protected void acquireRead() {
        assert Thread.holdsLock(this) : "Usage error: Cannot acquire lock unless the monitor is held!";
        readerCount++;
    }

    protected void releaseRead() {
        assert Thread.holdsLock(this) : "Usage error: Cannot release lock unless the monitor is held!";
        if (-- readerCount == 0) notify();
    }

    protected void acquireWrite() {
        assert Thread.holdsLock(this) : "Usage error: Cannot acquire lock unless the monitor is held!";
        if (readerCount > 0) {
            // like {@code synchronized}, we do not throw an exception on interrupt
            boolean intr = Thread.interrupted();
            try {
                do try {
                    wait();
                } catch (InterruptedException ignored) {
                    intr = true;
                } while (readCount > 0);
            } finally {
                if (intr) Thread.currentThread().interrupt();
            }
            intr = true;
        }
    }

    protected void releaseWrite() {
        assert Thread.holdsLock(this) : "Usage error: Cannot release lock unless the monitor is held!";
        notify();
    }
}
```

#### Usage: read lock

Here's how we use the read lock:

```java
public final class MyExample extends ReadWriteLockable {
    // ...

    public void example1() {
        // example of doing work under the read lock.
        synchronized (this) {
            acquireRead();
        }
        try {
            // do some read-locked work here
        } finally {
            synchronized (this) {
                releaseRead();
            }
        }
    }

    // ...
}
```

Notice that we have separate ```synchronized``` blocks for the acquire and release operations.  This is essential to allow for shared operation; only one thread can hold the monitor at a time so if we don't release it while we do our work, the lock cannot be shared and all our effort is for nothing.

#### Usage: write lock

Using the write lock is somewhat different:

```java
public final class MyExample extends ReadWriteLockable {
    // ...

    public void example2() {
        // example of doing work under the write lock.
        synchronized (this) {
            acquireWrite();
            try {
                // do some write-locked work here
            } finally {
                releaseWrite();
            }
        }
    }

    // ...
}
```

In this case, notice that we do all the work under a single ```synchronized``` lock; this is how our exclusive locking behavior is implemented, using Java's own exclusive locking behavior.

#### Usage: lock downgrading

Sometimes with a readers/writer lock, you want to atomically downgrade a write lock to a read lock for various reasons.  Using this implementation, this is how it would be accomplished safely:

```java
public final class MyExample extends ReadWriteLockable {
    // ...

    public void example3() {
        // example of doing some work under the write lock, and then downgrading to a read lock for the rest of the work.
        synchronized (this) {
            acquireWrite();
            try {
                // do some write-locked work here
            } finally {
                releaseWrite();
            }
            acquireRead();
        }
        try {
            // do some read-locked work here
        } finally {
            synchronized (this) {
                releaseRead();
            }
        }
    }

    // ...
}
```

The downgrade is <em>atomic</em>, meaning that any updates performed under the write-locked portion will be visible unchanged under the read-locked portion; in other words, there is no observable period of time where neither the read nor write lock is held in transition from the write-locked state to the read-locked state.

Note that in the success case, the ```releaseWrite()``` does not actually serve a useful purpose: it just triggers a spurious ```notify()```, causing one waiting writer (if there are any) to wake up and immediately sleep again.  Nevertheless, it cannot be removed, otherwise an exception thrown in the write-locked work section would cause any waiting writers to remain stuck, possibly indefinitely.

### Variations

There are numerous variations on this style of lock; for example, some implementations require that no additional readers be allowed if there is a writer waiting for a turn to lock, or that all lock requests must strictly be queued in order.  The implementation I've given here will block all writers until all readers have released the lock; that is, it strongly prefers readers over writers, which is appropriate for our use case, but maybe not for yours.

It is possible to use some bits of the counter (or a second counter) to track the number of waiting writers, and have any additional readers wait in this case.  Such a modification, while useful for certain use cases, might allow deadlocks in cases where the simple implementation does not.

### Acknowledgements

Thanks to my colleague Richard Opálka, and to ```##java``` denizen ```yawcat```, for finding bugs in the original workup.

