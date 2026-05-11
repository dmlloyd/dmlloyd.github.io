---
layout: post-series
series: The new reflection
title: The new reflection - VarHandle fundamentals
description: An introduction to VarHandle covering motivation, acquisition, and basic access modes.
categories: []
author: dmlloyd
date: 2026-05-11 09:50:00
---
## Recap

In the [previous posts](/posts/2026-05-07-the-new-reflection-advanced/), we covered method handles in depth: from basic lookup and invocation through advanced composition and control flow combinators.

Now we turn to a closely related abstraction: <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/VarHandle.html" target="javadoc">`VarHandle`</a>. Where `MethodHandle` gives you indirect access to _methods_, `VarHandle` gives you indirect access to _variables_ - with fine-grained control over memory ordering and atomic operations.

## Why VarHandle?

Before Java 9, if you needed anything beyond plain field access, your options were limited and each had significant drawbacks.

### `volatile`

The `volatile` keyword provides visibility guarantees and prevents reordering, but it's all-or-nothing: every access to a `volatile` field pays the full cost of sequential consistency. There is no way to perform a plain (cheap) read of a `volatile` field when you know the ordering guarantee isn't needed, nor is there any way to perform an atomic compare-and-set.
Also, other weaker ordering paradigms like acquire/release are not directly supported by `volatile`.

### `java.util.concurrent.atomic`

The `Atomic*` classes (`AtomicInteger`, `AtomicReference`, etc.) provide atomic operations, but they require a _wrapper object_. This adds an extra level of indirection and an extra allocation which is sometimes not acceptable.

### `sun.misc.Unsafe`

`Unsafe` can do nearly everything: atomic operations, fine-grained ordering, direct memory access. But it is, as the name suggests, _unsafe_. It performs no type checking, no bounds checking, and no access control. It requires raw field offsets obtained through reflection. And it is an internal API that is not portable and is being progressively restricted, and will eventually be removed.

### VarHandle: the safe middle ground

`VarHandle`, introduced in Java 9, provides the flexibility of `Unsafe` with the safety of a proper API:

* **Type-safe**: operations are signature-polymorphic, just like `MethodHandle`, so you get compile-time type checking without boxing
* **Access-controlled**: acquisition goes through `Lookup`, so access is checked once at creation time
* **Fine-grained ordering**: you choose the memory ordering strength for each individual access
* **No wrapper overhead**: operates directly on a field, array element, or memory segment

## What is a VarHandle?

A `VarHandle` is a typed, dynamically-checked reference to a _variable_. A variable, in this context, can be:

* An instance field of a class
* A static field of a class
* An element of an array
* An element of an NIO buffer
* A region of a <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/MemorySegment.html" target="javadoc">`MemorySegment`</a> (for off-heap or structured memory access on recent JDKs only)

Once you have a `VarHandle`, you can perform multiple kinds of operations on the same underlying variable. These operations are called _access modes_, and they range from plain reads and writes all the way to atomic compare-and-set and bitwise operations.

Like `MethodHandle`, `VarHandle` uses signature polymorphism. The access mode methods (like `get`, `set`, `compareAndSet`, etc.) accept and return values whose types are determined by the variable being accessed and the specific access mode.

## Acquiring a VarHandle

### Instance fields via `Lookup`

The most common way to acquire a `VarHandle` is through a <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findVarHandle(java.lang.Class,java.lang.String,java.lang.Class)" target="javadoc">`Lookup`</a>. For an instance field:

```java
class Counter {
    private int count;

    private static final VarHandle countHandle;

    static {
        try {
            MethodHandles.Lookup lookup = MethodHandles.lookup();
            countHandle = lookup.findVarHandle(Counter.class, "count", int.class);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
}
```

The three arguments are the declaring class, the field name, and the field type. As with method handle lookups, the access check happens at creation time.

### Avoiding the try/catch with `ConstantBootstraps`

As we covered in the [previous post](/posts/2026-05-07-the-new-reflection-advanced/), <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/ConstantBootstraps.html#fieldVarHandle(java.lang.invoke.MethodHandles.Lookup,java.lang.String,java.lang.Class,java.lang.Class,java.lang.Class)" target="javadoc">`ConstantBootstraps.fieldVarHandle`</a> provides a cleaner alternative that throws no checked exceptions:

```java
class Counter {
    private int count;

    private static final VarHandle countHandle = ConstantBootstraps.fieldVarHandle(
        MethodHandles.lookup(),
        "count",
        VarHandle.class,
        Counter.class,
        int.class
    );
}
```

This is the preferred pattern for field `VarHandle` acquisition. It eliminates the static initializer block entirely.

### Static fields

For static fields, use <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findStaticVarHandle(java.lang.Class,java.lang.String,java.lang.Class)" target="javadoc">`findStaticVarHandle`</a>:

```java
countHandle = lookup.findStaticVarHandle(Counter.class, "globalCount", int.class);
```

`VarHandle` instances for static fields do not take a receiver argument in their access mode methods.

### Array elements

For array element access, use <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#arrayElementVarHandle(java.lang.Class)" target="javadoc">`MethodHandles.arrayElementVarHandle`</a>:

```java
VarHandle arrayHandle = MethodHandles.arrayElementVarHandle(int[].class);
```

An array `VarHandle` takes two leading parameters in its access modes: the array itself and the index. So in this case, because it is an `int[]` typed array, `get` becomes `(int[], int)int` and `set` becomes `(int[], int, int)void`.

## Access mode categories

Every `VarHandle` supports some or all of five categories of access mode. Not every `VarHandle` supports every mode - for example, a handle to a `final` field only supports reads.

Some of these operations imply concurrency modes (like "opaque" or "acquire") which are beyond the scope of this post.

### Read access

Read-only access modes retrieve the current value of the variable:

* `get` - plain read
* `getVolatile` - volatile read
* `getOpaque` - opaque read (atomicity, but no ordering)
* `getAcquire` - acquire read (pairs with a release write)

### Write access

Write-only access modes set the value of the variable:

* `set` - plain write
* `setVolatile` - volatile write
* `setOpaque` - opaque write
* `setRelease` - release write (pairs with an acquire read)

### Atomic update

These modes atomically read the old value, compare it, and conditionally write a new value:

* `compareAndSet` - returns `boolean` (success/failure)
* `compareAndExchange*` - returns the _witness value_ (the value that was actually found), with an optional weaker concurrency mode
* `weakCompareAndSet*` - may spuriously fail, but can be faster in certain circumstances (see the documentation for details); also supports even more optional weaker concurrency modes

Concurrency algorithms are beyond the scope of this post, but it's worth noting that `compareAndExchange` has an advantage over `compareAndSet` and its variants, not only because of the additional concurrency modes available on that method but because it does the compare-and-set and a combined read in one operation:

```java
// read the old value once
String oldVal = myHandle.getVolatile(this);
String newVal = oldVal + " - Something new!";
// give it a try
String witness = myHandle.compareAndExchange(this, oldVal, newVal);
while (witness != oldVal) {
    // continue retrying until we do it
    oldVal = witness;
    newVal = oldVal + " - Something new!";
    witness = myHandle.compareAndExchange(this, oldVal, newVal);
}
```

### Numeric atomic update

For numeric variable types (`int`, `long`, `float`, `double`, and their wrappers), these modes atomically read and modify:

* `getAndAdd*` - atomic addition, returns the old value, with an optional concurrency mode

### Bitwise atomic update

For integer variable types (`int`, `long`), atomic bitwise operations:

* `getAndBitwiseOr*`, `getAndBitwiseAnd*`, `getAndBitwiseXor*` - each returns the old value, with an optional concurrency mode

Each of the atomic modes also comes in `Acquire` and `Release` variants.

### Checking support

You can check at run time whether a particular access mode is supported:

```java
if (countHandle.isAccessModeSupported(VarHandle.AccessMode.COMPARE_AND_SET)) {
    // safe to use compareAndSet on this handle
}
```

## Basic access: plain and volatile

To get started with `VarHandle`, the two most important access modes are _plain_ and _volatile_. These correspond to ordinary field access and `volatile` field access, respectively.

### Plain access

Plain access via `get` and `set` has the same semantics as reading or writing a non-volatile field. There are no special ordering guarantees beyond what the Java Memory Model provides for normal (non-volatile) accesses.

```java
class Counter {
    private int count;

    private static final VarHandle countHandle = ConstantBootstraps.fieldVarHandle(
        MethodHandles.lookup(), "count", VarHandle.class,
        Counter.class, int.class);

    public int getCount() {
        return (int) countHandle.get(this);
    }

    public void setCount(int value) {
        countHandle.set(this, value);
    }
}
```

Note the signature polymorphism at work: `get` takes the receiver (`this`, of type `Counter`) and returns an `int`, matching the field type. No boxing, no array wrapping.

### Volatile access

Volatile access via `getVolatile` and `setVolatile` provides the same guarantees as reading or writing a `volatile` field: full sequential consistency and a happens-before relationship between a volatile write and a subsequent volatile read of the same variable.

```java
public int getCountVolatile() {
    return (int) countHandle.getVolatile(this);
}

public void setCountVolatile(int value) {
    countHandle.setVolatile(this, value);
}
```

One key insight is that the _same field_ - declared without the `volatile` keyword - can be accessed with _different_ ordering strengths depending on the situation.

## Next...

In the final post of this series, we'll bring `VarHandle` and `MethodHandle` together by showing how to extract method handles from a `VarHandle` and compose them using the combinators we covered earlier.
