---
layout: post-series
series: The new reflection
title: The new reflection - Intermediate use cases
description: Intermediate method handle usage covering lookup factories, handle adaptations, and API design.
categories: []
author: dmlloyd
date: 2026-04-14 10:36:00
---
## Recap

In the [previous post](/posts/2024-11-08-the-new-reflection/), I covered the basics of `MethodHandle`: what it is, how signature polymorphism works, the difference between exact and inexact invocation, how to handle exceptions, and how to acquire a `Lookup`.

In this post, we'll put that foundation to work. We'll look at the different kinds of members you can look up, how to adapt method handles to fit different call sites, and how to design clean APIs around them.

## Lookup factory methods

As stated in the previous installment, the <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html" target="javadoc">`MethodHandles.Lookup`</a> class provides a number of factory methods for creating method handles that represent different kinds of member access. Each factory method performs an access check at creation time and returns a `MethodHandle` that can be used freely thereafter with no further checks. Let's look at specifics around how to acquire handles for each kind of member.

### Looking up methods

The two most common factories are <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findVirtual(java.lang.Class,java.lang.String,java.lang.invoke.MethodType)" target="javadoc">`findVirtual`</a> and <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findStatic(java.lang.Class,java.lang.String,java.lang.invoke.MethodType)" target="javadoc">`findStatic`</a>. Both take the declaring class, the method name, and a `MethodType` describing the parameter and return types.

```java
MethodHandles.Lookup lookup = MethodHandles.lookup();

// String.length(): an instance method taking no arguments, returning int
MethodHandle length = lookup.findVirtual(String.class, "length",
    MethodType.methodType(int.class));

// Integer.parseInt(String): a static method taking String, returning int
MethodHandle parseInt = lookup.findStatic(Integer.class, "parseInt",
    MethodType.methodType(int.class, String.class));
```

Note that a handle obtained via `findVirtual` has an extra leading parameter: the receiver (the object the method is called on). So the `length` handle above has the effective type `(String)int`, not `()int`.

```java
int len = (int) length.invokeExact("hello"); // returns 5
int val = (int) parseInt.invokeExact("42"); // returns 42
```

### Looking up constructors

The <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findConstructor(java.lang.Class,java.lang.invoke.MethodType)" target="javadoc">`findConstructor`</a> factory creates a handle that allocates and initializes a new instance. The `MethodType` you pass describes the constructor's _parameters_ only; the return type must always be `void` (the factory itself knows the return type is the declaring class).

```java
// new ArrayList(int initialCapacity)
MethodHandle newArrayList = lookup.findConstructor(ArrayList.class,
    MethodType.methodType(void.class, int.class));

ArrayList<?> list = (ArrayList<?>) newArrayList.invokeExact(16);
```

Note that the return type of the handle is always going to be the type of object that was created, even though you gave `void` as the return type to the factory method.

### Looking up fields

Field access can be exposed as getter and setter method handles. There are four variants:

* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findGetter(java.lang.Class,java.lang.String,java.lang.Class)" target="javadoc">`findGetter`</a> / <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findSetter(java.lang.Class,java.lang.String,java.lang.Class)" target="javadoc">`findSetter`</a> for instance fields
* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findStaticGetter(java.lang.Class,java.lang.String,java.lang.Class)" target="javadoc">`findStaticGetter`</a> / <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#findStaticSetter(java.lang.Class,java.lang.String,java.lang.Class)" target="javadoc">`findStaticSetter`</a> for static fields

```java
// Assume: class Point { int x; int y; }
MethodHandle getX = lookup.findGetter(Point.class, "x", int.class);
MethodHandle setX = lookup.findSetter(Point.class, "x", int.class);

Point p = new Point();
setX.invokeExact(p, 10);
int x = (int) getX.invokeExact(p); // returns 10
```

An instance getter has type `(DeclaringClass)FieldType`, while an instance setter has type `(DeclaringClass,FieldType)void`. Static variants have no leading receiver parameter.

## Bridging from old-style reflection

If you already have a `java.lang.reflect.Method`, `Field`, or `Constructor` instance, you can convert it to a `MethodHandle` using the `unreflect` family of methods:

* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#unreflect(java.lang.reflect.Method)" target="javadoc">`unreflect(Method)`</a>
* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#unreflectGetter(java.lang.reflect.Field)" target="javadoc">`unreflectGetter(Field)`</a> / <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#unreflectSetter(java.lang.reflect.Field)" target="javadoc">`unreflectSetter(Field)`</a>
* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#unreflectConstructor(java.lang.reflect.Constructor)" target="javadoc">`unreflectConstructor(Constructor)`</a>

This is useful when you're working with frameworks that give you reflection objects, or when you need annotation information from a `Method` before converting it to a handle.
These methods perform a one-time access check which also considers the `accessible` status of the source reflection object.

### Going the other way

You can also do the opposite, and acquire a reflection object from a method handle.
This is accomplished with a single method: <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#reflectAs(java.lang.Class,java.lang.invoke.MethodHandle)" target="javadoc">`unreflectAs`</a>.
This can be quite useful in certain specific scenarios, such as creating a `Method` from a class file `MethodHandle` constant (an advanced use case).

Note that there are several restrictions on the method handle in question; in particular, it has to be what is known as a <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandleInfo.html#directmh">direct</a> method handle.
A method handle is direct when there are no transformations associated with it (which will be described in the next section).

## Adapting method handles

Once you have a method handle, you often need to adjust it to fit a different call site, to preestablish arguments, or to adapt the return value or argument types. The `MethodHandle` and `MethodHandles` classes provide several methods for this purpose. Each returns a new `MethodHandle` with a modified type; the original is unchanged.

### Binding the receiver

The <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandle.html#bindTo(java.lang.Object)" target="javadoc">`bindTo`</a> method fixes the first argument of a method handle to a specific object, removing it from the parameter list.
This is most commonly used to bind the receiver of a virtual method:

```java
MethodHandle length = lookup.findVirtual(String.class, "length",
    MethodType.methodType(int.class));
// length has type (String)int

MethodHandle helloLength = length.bindTo("hello");
// helloLength has type ()int

int len = (int) helloLength.invokeExact(); // returns 5
```

While binding the receiver is the most common usage, this method generally works for any case where you want to bind the first argument of a method.

### Inserting arguments

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#insertArguments(java.lang.invoke.MethodHandle,int,java.lang.Object...)" target="javadoc">`MethodHandles.insertArguments`</a> is the generalized form of `bindTo`. It lets you fix one or more arguments at _any_ position, not just the first:

```java
// Integer.compare(int, int) -> int
MethodHandle compare = lookup.findStatic(Integer.class, "compare",
    MethodType.methodType(int.class, int.class, int.class));
// compare has type (int,int)int

MethodHandle compareToZero = MethodHandles.insertArguments(compare, 1, 0);
// compareToZero has type (int)int — second argument is fixed to 0

int result = (int) compareToZero.invokeExact(5); // returns 1 (5 > 0)
```

### Dropping arguments

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#dropArguments(java.lang.invoke.MethodHandle,int,java.lang.Class...)" target="javadoc">`MethodHandles.dropArguments`</a> is the inverse: it _adds_ parameters to a handle's type that are silently ignored when invoked. This is useful when you need a handle to conform to a specific type signature but don't need all the arguments:

```java
MethodHandle parseInt = lookup.findStatic(Integer.class, "parseInt",
    MethodType.methodType(int.class, String.class));
// parseInt has type (String)int

MethodHandle ignoringFirst = MethodHandles.dropArguments(parseInt, 0, Object.class);
// ignoringFirst has type (Object,String)int — first argument is ignored

int val = (int) ignoringFirst.invokeExact((Object) "unused", "42"); // returns 42
```

### Reordering arguments

<a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#permuteArguments(java.lang.invoke.MethodHandle,java.lang.invoke.MethodType,int...)" target="javadoc">`MethodHandles.permuteArguments`</a> is the most powerful of the basic transformations.
It takes a target handle, a new `MethodType`, and an array of indices that maps each argument of the new type to an argument of the target. This lets you reorder, duplicate, or drop arguments:

```java
// Suppose we have a handle with type (String, int)void
// and we want to swap the arguments to get (int, String)void:
MethodHandle swapped = MethodHandles.permuteArguments(original,
    MethodType.methodType(void.class, int.class, String.class),
    1, 0);  // new arg 0 -> target arg 1, new arg 1 -> target arg 0
```

## Utility method handles

The `MethodHandles` class provides several factory methods that produce simple, reusable handles. These may seem trivial on their own, but they become essential building blocks when composing method handles (which we will cover in the next post).

* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#constant(java.lang.Class,java.lang.Object)" target="javadoc">`MethodHandles.constant(Class, Object)`</a> returns a handle that always returns the given constant value, ignoring any arguments (it takes no parameters).

* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#identity(java.lang.Class)" target="javadoc">`MethodHandles.identity(Class)`</a> returns a handle that takes one argument and returns it unchanged. Its type is `(T)T`.

* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#zero(java.lang.Class)" target="javadoc">`MethodHandles.zero(Class)`</a> returns a handle that takes no arguments and returns the zero value for the given type (`0` for numeric types, `false` for `boolean`, `null` for reference types).

* <a href="https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/MethodHandles.html#empty(java.lang.invoke.MethodType)" target="javadoc">`MethodHandles.empty(MethodType)`</a> returns a handle that accepts whatever arguments the given type describes but ignores them all and returns the zero value for its return type.

These are useful whenever you need a "do nothing" or "return a fixed value" handle as a placeholder or default in a larger composition.

## API design with method handles

Method handles are a powerful implementation tool, but they require some care when used as part of an API.

### Keep handles private

A `MethodHandle` is an unrestricted capability: once you have one, you can call it with no further access checks.
This means you should treat method handles the way you treat other private data: store them on `private` fields and do not expose them to callers.

```java
public class Widgets {
    private static final MethodHandle MH_CREATE;

    static {
        try {
            MethodHandles.Lookup lookup = MethodHandles.lookup();
            MH_CREATE = lookup.findConstructor(Widget.class,
                MethodType.methodType(void.class, String.class));
        } catch (NoSuchMethodException | IllegalAccessException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    public static Widget create(String name) {
        try {
            return (Widget) MH_CREATE.invokeExact(name);
        } catch (RuntimeException | Error e) {
            throw e;
        } catch (Throwable t) {
            throw new UndeclaredThrowableException(t);
        }
    }
}
```

### Adapt once, store, reuse

If you need to adapt a method handle (using `asType`, `bindTo`, `insertArguments`, etc.), do it once during initialization and store the result.
Re-adapting on every call defeats the purpose of hoisting access checks to creation time and prevents the JIT from treating the handle as a constant.

### Guard your Lookup

When accepting a `Lookup` from another module (e.g., as a parameter to a framework entry point), use it immediately to look up whatever you need, and then discard it.
Do not store a foreign `Lookup` on a field unless absolutely necessary.
This limits the window during which the access capability is live and avoids accidentally leaking it.

## Next...

In the next post, we'll look at the more advanced combinators for _composing_ method handles: building complex behavior out of simple parts using `filterArguments`, `guardWithTest`, `catchException`, and more.
