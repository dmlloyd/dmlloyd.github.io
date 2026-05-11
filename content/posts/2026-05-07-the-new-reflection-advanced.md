---
layout: post
title: The new reflection - Advanced method handles
description: Advanced method handle composition, control flow combinators, ConstantBootstraps, and performance.
categories: []
author: dmlloyd
date: 2026-05-07 09:00:00
---
## Recap

In the [previous post](/posts/2026-04-14-the-new-reflection-intermediate/), we covered the lookup factory methods, basic adaptations like `bindTo` and `insertArguments`, and some principles of API design with method handles.

In this post, we'll explore the more advanced combinators that let you _compose_ method handles into pipelines of behavior: transforming arguments and return values, branching on conditions, handling exceptions, and more.

## Composing method handles

The `MethodHandles` class provides a family of static methods that combine one or more method handles into a new one. These are the method handle equivalent of function composition, and they're what turn method handles from a simple reflection replacement into a genuinely powerful abstraction.

### Transforming arguments with `filterArguments`

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#filterArguments(java.lang.invoke.MethodHandle,int,java.lang.invoke.MethodHandle...)" target="javadoc">`MethodHandles.filterArguments`</a> lets you transform one or more arguments before they reach the target handle. Each filter is a unary handle: it takes one argument and returns a transformed value that replaces the original argument at its position.

```java
// Target: a method that takes an int
// Filter: Integer.parseInt(String) -> int
// Result: a method that takes a String (which gets parsed to int first)

MethodHandle target = ...;  // type (int)void
MethodHandle filter = lookup.findStatic(Integer.class, "parseInt",
    MethodType.methodType(int.class, String.class));

MethodHandle filtered = MethodHandles.filterArguments(target, 0, filter);
// filtered has type (String)void
filtered.invokeExact("42");  // calls target with (int) 42
```

The second parameter is the position at which to start applying filters. You can supply multiple filters to transform consecutive arguments.

### Transforming the return value with `filterReturnValue`

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#filterReturnValue(java.lang.invoke.MethodHandle,java.lang.invoke.MethodHandle)" target="javadoc">`MethodHandles.filterReturnValue`</a> applies a transformation to the result _after_ the target handle completes. The filter handle must accept exactly the return type of the target and may return any type.

```java
// Target: String.length() -> int
// Filter: a handle that boxes an int to Integer
// Result: String -> Integer

MethodHandle length = lookup.findVirtual(String.class, "length",
    MethodType.methodType(int.class));
MethodHandle box = lookup.findStatic(Integer.class, "valueOf",
    MethodType.methodType(Integer.class, int.class));

MethodHandle boxedLength = MethodHandles.filterReturnValue(length, box);
// boxedLength has type (String)Integer
Integer len = (Integer) boxedLength.invokeExact("hello"); // returns (Integer) 5
```

### Computing a prefix value with `foldArguments`

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#foldArguments(java.lang.invoke.MethodHandle,java.lang.invoke.MethodHandle)" target="javadoc">`MethodHandles.foldArguments`</a> is a bit more subtle. It calls a _combiner_ handle on some prefix of the arguments, and then calls the target handle with the combiner's return value _prepended_ to the original arguments.

This is useful when you need to compute a value from the arguments and pass both the computed value _and_ the original arguments to the target.

```java
// Combiner: computes a hash code from a String
// Target: takes (int hashCode, String value) and does something with both

MethodHandle combiner = lookup.findVirtual(String.class, "hashCode",
    MethodType.methodType(int.class));
// combiner has type (String)int

MethodHandle target = ...;  // type (int, String)void

MethodHandle folded = MethodHandles.foldArguments(target, combiner);
// folded has type (String)void
// When called: combiner runs first on the String, producing an int;
// then target is called with (int, String)
```

### Collecting arguments with `collectArguments`

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#collectArguments(java.lang.invoke.MethodHandle,int,java.lang.invoke.MethodHandle)" target="javadoc">`MethodHandles.collectArguments`</a> is a generalization that replaces a single argument of the target at a given position with the result of calling a collector handle on some arguments. Unlike `foldArguments`, the collector's arguments are _consumed_ and do not appear in the resulting handle's type.

```java
// Target: takes (String, int)void
// Collector at position 1: Integer.parseInt(String) -> int
// Result: takes (String, String)void - the second String is parsed to int

MethodHandle collected = MethodHandles.collectArguments(target, 1, parseInt);
// When called with ("hello", "42"):
//   parseInt("42") -> 42
//   target("hello", 42)
```

## Spreading and collecting arrays

Sometimes you need to bridge between array-based and positional argument passing.

### Spreading: array to positional

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandle.html#asSpreader(java.lang.Class,int)" target="javadoc">`MethodHandle.asSpreader`</a> converts a handle that takes N positional arguments into one that takes an array and spreads it across those positions:

```java
MethodHandle compare = lookup.findStatic(Integer.class, "compare",
    MethodType.methodType(int.class, int.class, int.class));
// compare has type (int, int)int

MethodHandle spread = compare.asSpreader(int[].class, 2);
// spread has type (int[])int

int result = (int) spread.invokeExact(new int[] { 3, 7 }); // returns -1
```

### Collecting: positional to array

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandle.html#asCollector(java.lang.Class,int)" target="javadoc">`MethodHandle.asCollector`</a> does the reverse: trailing positional arguments are gathered into an array before calling the target.

```java
MethodHandle target = ...;  // type (String, Object[])void

MethodHandle collecting = target.asCollector(Object[].class, 3);
// collecting has type (String, Object, Object, Object)void
```

The related <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandle.html#asVarargsCollector(java.lang.Class)" target="javadoc">`asVarargsCollector`</a> goes a step further, making the handle behave like a varargs method: it will accept _any_ number of trailing arguments and automatically collect them into an array.

## Control flow combinators

One of the most powerful aspects of method handles is the ability to express control flow - branching, exception handling, and cleanup - as composed method handles rather than wrapper methods.

### Conditional execution with `guardWithTest`

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#guardWithTest(java.lang.invoke.MethodHandle,java.lang.invoke.MethodHandle,java.lang.invoke.MethodHandle)" target="javadoc">`MethodHandles.guardWithTest`</a> is the method handle equivalent of `if`/`else`. It takes three handles:

* **test**: a handle returning `boolean`
* **target**: called when the test returns `true`
* **fallback**: called when the test returns `false`

All three must accept the same argument types. The test's arguments are passed to whichever branch is selected.

```java
MethodHandle isNull = ...;   // type (Object)boolean
MethodHandle toString = ...; // type (Object)String
MethodHandle fallback = MethodHandles.constant(String.class, "null");
fallback = MethodHandles.dropArguments(fallback, 0, Object.class);

MethodHandle safeToString = MethodHandles.guardWithTest(isNull, fallback, toString);
// safeToString has type (Object)String
```

### Exception handling with `catchException`

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#catchException(java.lang.invoke.MethodHandle,java.lang.Class,java.lang.invoke.MethodHandle)" target="javadoc">`MethodHandles.catchException`</a> wraps a target handle in a try/catch. If the target throws an exception of the specified type, the handler is called instead. The handler receives the caught exception as its first argument, followed by the original arguments.

```java
MethodHandle risky = ...;   // type (String)int, might throw NumberFormatException
MethodHandle handler = ...; // type (NumberFormatException, String)int

MethodHandle safe = MethodHandles.catchException(risky,
    NumberFormatException.class, handler);
// safe has type (String)int - exception is caught and handled
```

### Cleanup with `tryFinally`

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#tryFinally(java.lang.invoke.MethodHandle,java.lang.invoke.MethodHandle)" target="javadoc">`MethodHandles.tryFinally`</a> ensures that a cleanup handle runs after the target, regardless of whether it completes normally or throws. The cleanup handle receives the thrown exception (or `null`), the result (or zero), and the original arguments.

```java
MethodHandle acquire = ...;  // type ()Resource
MethodHandle cleanup = ...;  // type (Throwable, Resource)void - null Throwable on success

MethodHandle managed = MethodHandles.tryFinally(acquire, cleanup);
```

## `ConstantBootstraps`

The <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/ConstantBootstraps.html" target="javadoc">`ConstantBootstraps`</a> class provides bootstrap methods originally intended for use with _constant dynamic_ (`condy`) entries in class files. However, several of its methods are useful to call directly because they wrap common operations in a way that avoids checked exceptions.

### Avoiding checked exceptions with `fieldVarHandle`

The standard way to acquire a `VarHandle` for a field via `Lookup` requires catching `NoSuchFieldException` and `IllegalAccessException`. The <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/ConstantBootstraps.html#fieldVarHandle(java.lang.invoke.MethodHandles.Lookup,java.lang.String,java.lang.Class,java.lang.Class,java.lang.Class)" target="javadoc">`ConstantBootstraps.fieldVarHandle`</a> method wraps this into a single call that throws only unchecked exceptions, making it ideal for inline field initialization:

```java
private static final VarHandle stateHandle = ConstantBootstraps.fieldVarHandle(
    MethodHandles.lookup(),
    "state",        // field name
    VarHandle.class, // the type of the constant being produced
    MyClass.class,  // declaring class
    int.class       // field type
);
```

We will explore `VarHandle` in detail in the next post, but the convenience of this pattern is worth noting here since it eliminates the need for a static initializer block with try/catch.

### Other useful bootstraps

* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/ConstantBootstraps.html#getStaticFinal(java.lang.invoke.MethodHandles.Lookup,java.lang.String,java.lang.Class,java.lang.Class)" target="javadoc">`getStaticFinal`</a> reads a `static final` field from a class, which is particularly useful for accessing constants like `BigInteger.ZERO` or `Boolean.TRUE` in a bootstrap context.
* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/ConstantBootstraps.html#enumConstant(java.lang.invoke.MethodHandles.Lookup,java.lang.String,java.lang.Class)" target="javadoc">`enumConstant`</a> looks up an enum constant by name, returning it as the appropriate enum type.
* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/ConstantBootstraps.html#invoke(java.lang.invoke.MethodHandles.Lookup,java.lang.String,java.lang.Class,java.lang.invoke.MethodHandle,java.lang.Object...)" target="javadoc">`invoke`</a> calls an arbitrary `MethodHandle` with given arguments and returns the result: useful for computing derived constants.

## Performance considerations

Method handles can be highly optimized by the JIT compiler, but this optimization depends on a few key practices.

### Store on `static final` fields

When a `MethodHandle` is stored on a `private static final` field, it is possible that the JIT can treat it as a compile-time constant. This enables _constant folding_: the JIT can see exactly what the handle does and potentially inline the entire chain of adaptations and compositions into the call site. This is the single most important optimization for method handle performance.

### Avoid unnecessary depth

Each combinator (`filterArguments`, `guardWithTest`, etc.) adds a layer of indirection. The JIT can often see through these layers, but only up to a point. If you find yourself chaining five or six combinators deep, consider whether a simple helper method would be clearer and more easily optimized.

### Prefer stable call profiles

A method handle stored on a `static final` field gives the JIT a _monomorphic_ call profile, meaning: it always sees the same handle. If a handle is loaded from a mutable field or passed as a parameter, the JIT may see different handles at different times, which can inhibit inlining. Where possible, arrange for handles to be stable constants.

## Next...

The next post will introduce <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/VarHandle.html" target="javadoc">`VarHandle`</a>: a related abstraction for fine-grained variable access with explicit control over memory ordering and atomic operations.
