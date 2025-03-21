---
layout: post
title: The new reflection - Basics
description: An introduction to using the advanced capabilities of member access since Java 9.
categories:
author: dmlloyd
date: 2024-11-08 09:03:29
---
## The new reflection

The basic reflection facility of Java historically amounts to using a `Class` object to acquire `Method`, `Constructor`, and/or `Field` objects. These objects are used primarily for invocation of a method or constructor (in the former two cases) or read/write access to a field (in the latter case). The mechanism is simple: each reflective object accepts an `Object` or array of `Object` and yields `void` or an `Object` with the result (depending on the operation).

Because primitives in Java are not `Object` instances, in order to pass or return values of primitive type, we rely on so-called "box objects", which are object classes whose purpose is to represent a primitive value as an `Object`.

These APIs are fairly useful for a number of use cases (including non-access cases like examining annotations), but come with several limitations:

* They're heavyweight, requiring boxing/unboxing of inputs and outputs
* Not type safe (generally operating on `Object` and/or `Object[]`)
* Access checking is generally done at every invocation/access point
* Hard to optimize
* Sometimes hard to use in conjunction with JPMS (modules)

## Method handles

In Java 7, a new class was introduced which was intended to overcome these limitations, called <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandle.html" target="javadoc">`MethodHandle`</a>.
This class, and its related classes in <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/package-summary.html" target="javadoc">`java.lang.invoke`</a>, are all designed to support a new and more efficient means
of indirect access to class members.
The initial motivating use case included support for the new `invokedynamic` JVM instruction as part of <a href="https://jcp.org/ja/jsr/detail?id=292" target="_blank">JSR 292</a>.

This efficiency is primarily achieved through a few specific mechanisms:

* Reducing the number of implementations needed (only one per call site signature versus one per target class)
* Hoisting access checks to creation time instead of usage time
* Usage of signature polymorphism (explained below)

In addition to acting as a replacement for a large number of reflection use cases, method handles are also the basis of lambdas and the [Foreign Function & Memory (FFM) API](https://openjdk.org/jeps/454), among other things.

### What is a method handle?

The `MethodHandle` class gives you some methods which can be used to perform an invocation. Of these, these two are the most commonly used:

* <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandle.html#invoke(java.lang.Object...)" target="javadoc">`MethodHandle.invoke(...)`</a>
* <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandle.html#invokeExact(java.lang.Object...)" target="javadoc">`MethodHandle.invokeExact(...)`</a>

These methods, and several others in this package, don't have a specific signature (that is, they have no particular number or type of argument). Rather, they are _signature polymorphic_.

### What is signature polymorphism?

In the Java language, a method is identified by the combination of its _name_ and its _argument types_. Because of this, we can provide many _overloads_ of a method with different argument types and quantity.

Signature polymorphism takes this concept one step farther.
When a method qualifies as being signature-polymorphic (meeting <a href="https://docs.oracle.com/javase/specs/jls/se23/html/jls-15.html#jls-15.12.3" target="spec">certain specific requirements</a>), it automatically has _every possible overload_. These methods are declared as though they return `Object` and accept `Object...` in order to satisfy Java language requirements, but they are treated specially such that no actual varargs array creation or primitive boxing/unboxing is done.

This means that all of these methods (and more) "exist" and can be directly invoked:

* `MethodHandle.invoke(int)`
* `MethodHandle.invoke(String,List,int[])`
* `MethodHandle.invoke(MethodHandle,double,Console)`
* `MethodHandle.invoke(Object[],Object[])`
* `MethodHandle.invoke(int,int,int,int,int,int,int,int,int,int,int,int)`

It doesn't stop there though. The Java _language_ may identify methods only by their name and argument types, but the Java _virtual machine_ additionally uses the return type to identify a method.
So, even though Java won't let you create an overload which differs only by return type, these kinds overloads will nevertheless exist as well - one for every _possible_ return type (which includes `void`).

The main benefit of this is that it is possible to call an `invoke` method with the exact argument types of the target without having to wrap them in an array or box the arguments, and with the exact return type of the target without having to unbox the result.
This in turn makes it easier for the compiler to optimize as well.

However, because of the aforementioned restrictions, the only place you will find this kind of method is in the `java.lang.invoke` package - specifically, `MethodHandle` and `VarHandle` (which will be covered later).

### How to call a signature polymorphic method

For the most part, you can call a signature polymorphic method like any other method.
However, when such a call is made, the exact method variant which is called is determined by the actual argument types and the inferred return type.

So for example, given `mh` of type `MethodHandle`, a _call site_ may look like this:

```java
int result = (int) mh.invokeExact(123, "Hello world!", Object.class);
```

In this case, the method being invoked is inferred to take three arguments, whose types are `int`, `String`, and `Class`, and whose return type is `int`, and so out of the infinite possible methods to call, this one (and only this one) is selected.

#### Ignoring the return value

This works fine as long as the arguments that you give correspond exactly to what the method expects, _and_ that the return type is also exactly what is expected.
However, consider the case where you don't care about the return value but you still want to call that same method.
With a normal Java invocation, you might try this:

```java
mh.invokeExact(123, "Hello world!", Object.class);
```

However, with signature polymorphic methods, this could give you a surprise: the selected method no longer has a return type of `int`, but of `void`, which can cause an error in some cases (explained up ahead).

To force the call to return the type you want, you have to cast the return type, and either assign it to an ignored variable, or otherwise do something with the result that is unambiguous in terms of type:

```java
var _ = (int) mh.invokeExact(123, "Hello world!", Object.class);
```

Now we've forced the Java compiler to establish that the called method returns `int`.
Note in this example we are using the relatively recent `_` feature, but on older Java versions, you can use a placeholder variable name like `ignored`.

#### Mismatching return or argument types

Another tricky case comes up when you want to pass in or return a value whose type is _different_ from the type on the method you want to call.

For example, imagine we want to call the same method as above, but we want the result to be stored in a `long`. If we just change our variable type to `long`, we know that the wrong method will be called; we specifically want the one which returns an `int`. We can solve this by ensuring that the cast matches the desired return type:

```java
long result = (int) mh.invokeExact(123, "Hello world!", Object.class);
```

The cast tells Java which method to call, and existing Java rules for widening values will ensure that the `int` result can be stored in the `long` variable.

The same thing is true with argument types: a cast can be used to specify the exact method we are interested in calling, when the actual argument value's type differs from the type of the corresponding parameter.

#### The double-cast

One last problem can occur when we specifically want to _narrow_ a result value (that is, cast it to a more specific type).
In such cases, we may end up having to cast the return value _twice_ (once to tell Java which method to invoke, and once to actually narrow the return value):

```java
String result = (String) (CharSequence) mh.invokeExact();
```

In this example we're calling something which is declared to return `CharSequence`, but we know (through some means) that it actually will always be an actual `String`.
Most IDEs will understand this construct and will not warn you about the seemingly redundant cast.

## Representing method types with `MethodType`

Every `MethodHandle` instance has a _method type_, represented by instances of <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodType.html" target="javadoc">`MethodType`</a> and accessible using the <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandle.html#type()" target="javadoc">`MethodHandle#type()`</a> accessor method.
This class encodes the argument types and return type (as `Class` instances) of a method, which suffices to identify the type of any given call site.
Every call site has an implied `MethodType` which goes along with it.

A `MethodHandle` may be transformed to have a different type by using the <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandle.html#asType(java.lang.invoke.MethodType)" target="javadoc">`MethodHandle#asType()`</a> method.
This method returns a new version of the given `MethodHandle`, but where each argument and the return type are based on those of the given `MethodType`.
The number of arguments in the new type must equal the number of arguments in the old type, and the conversion must be valid.
For example, if you have a method handle whose return type is `String`, and you try to convert it to one which returns an `int`, an exception is thrown.
However, converting a method handle which returns `String` to one which returns `CharSequence` is perfectly valid.

## Exact versus inexact invocation

A method handle may be invoked _exactly_ or _inexactly_.
Invoking a method handle _exactly_ is done via the `MethodHandle.invokeExact(...)` method.
When performing an `exact` invocation, the call site's implied type and the type of the `MethodHandle` _must_ be _exactly_ identical (even the return type), or an exception will be thrown at run time.

_Inexact_ invocation via the `MethodHandle.invoke(...)` method is much more flexible.
Rather than requiring the method handle's type to _exactly_ match, an _inexact_ invocation works on any method handle which can be reasonably converted to the call site type (as if by calling `MethodHandle.asType(callSiteType)` with the exact type of the call site).
However, this flexibility comes at a cost, since the argument types have to be checked at run time; this may be unnecessary in many cases for _exact_ invocations.

If a method handle is only going to be used once, then an _inexact_ call via `invoke(...)` is usually going to be fine.
Otherwise, it is usually preferable to use `invokeExact(...)`.
One common strategy is to adapt an original `MethodHandle` to the expected call site type using `asType()`, and store the result in some place where it can be reused thereafter.

## Handling exceptions

Unfortunately, the invocation methods on `MethodHandle` are declared to throw `Throwable`.
This is because a `MethodHandle` can refer to anything in the JVM that can be called, and those things in turn can be declared to throw any kind or number of exceptions.

To cope with this, you may be tempted to wrap the `Throwable` with some kind of `RuntimeException` subclass and rethrow it unconditionally. __Do not do this__!

Instead, use this pattern _always_:

```java
private void invokeIt(MethodHandle handle, int foo, String bar) {
    try {
        int ignored = (int) handle.invokeExact(foo, bar);
    } catch (RuntimeException | Error e) {
        throw e;
    } catch (Throwable t) {
        throw new UndeclaredThrowableException(t);
    }
}
```

If the method handle in question has some specific checked exceptions that can be thrown, add those to the list:

```java
private void invokeIt(MethodHandle handle, int foo, String bar) throws SpecificException {
    try {
        int ignored = (int) handle.invokeExact(foo, bar);
    } catch (RuntimeException | Error | SpecificException e) {
        throw e;
    } catch (Throwable t) {
        throw new UndeclaredThrowableException(t);
    }
}
```

## Acquiring a `MethodHandle`: `Lookup`

A `MethodHandle` is not useful unless you can somehow acquire one.
In order to acquire a `MethodHandle`, you need an instance of <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html" target="javadoc">`MethodHandles.Lookup`</a>.

A `Lookup` instance provides factory methods which can create method handles representing:

* Static and instance (virtual) methods
* "Special" methods e.g. `super.foo()`
* Static and instance field getters and setters
* Constructors

A `Lookup` functions as an access key to a given _lookup class_, which is the class that is associated with the `Lookup` instance. The lookup class can be returned by calling the `lookupClass()` getter method on a `Lookup` instance.

The access power of the `Lookup` depends not only on the lookup class, but also on its <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#lookupModes()" target="javadoc">_lookup mode_</a>.
The lookup mode determines which access levels are accessible by the `Lookup`.
For example, a `Lookup` with `PRIVATE` access may be used to access any `private` member that is accessible from the lookup class. However a `Lookup` which lacks `PRIVATE` access may not access any `private` members, even those which would otherwise be accessible from the lookup class.
A lookup with reduced lookup modes may be created by calling the <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#dropLookupMode(int)" target="javadoc"> `dropLookupMode(int)`</a> method with the specific mode to drop.

There are several possible ways to acquire a `Lookup`.
The best strategy to use depends on use case.

### The public `Lookup`

The public `Lookup` is a `Lookup` which can access any `public` member on any `public` class.
It is a singleton which can be acquired by calling <a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandles.html#publicLookup()" target="javadoc">`MethodHandles.publicLookup()`</a>.
No special module flags or privileges are required to acquire and use this `Lookup`. The lookup class of the public `Lookup` is `Object.class`.
This type of `Lookup` is most suitable for use implementations of APIs which only require access to `public` members.

### Full privilege `Lookup`

A full-privilege lookup is available to every class by way of the
<a href="https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandles.html#lookup()" target="javadoc">`MethodHandles.lookup()`</a> method, and in fact this is the most common way to acquire a `Lookup` other than the public `Lookup` instance.

This method is `static` and does not accept any arguments.
The lookup class of the returned `Lookup` is always that of the calling class,
and the returned `Lookup` always has full privileges to that class (including access to all `private` fields, methods, and constructors).
Therefore it is very important that this instance be _secured_ by the caller.
Specifically, *store the instance only on private fields* and *do not share the instance with untrusted APIs* without reducing its access mode first.

This type of `Lookup` is suitable for a number of use cases:

* When explicitly granting permissions to other frameworks/APIs
* When accessing members within the same module and package
* As a seed for gaining access to other modules

### Private access `Lookup`

Despite what it may seem based on what has been said so far, it is in fact possible (since Java 9) to acquire a `Lookup` that allows you to gain access to another class without the class explicitly providing you with its full-privilege `Lookup`.
This access however is mediated by Java access controls, specifically those relating to modules.
It also requires an existing full-privilege lookup to act as a seed.

To acquire the private `Lookup`, the instance method [`lookup.privateLookupIn(Class<?>)`](https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/MethodHandles.html#privateLookupIn(java.lang.Class,java.lang.invoke.MethodHandles.Lookup)) is called on the seed `Lookup`.
This method performs an access control check which is based on the lookup class of the original `Lookup` to determine whether that class is allowed to access the target class.
This check will pass if the target class is in a module that is `open`, or the package of the target class is `open`, or the package is `open` specifically to the module of the original lookup class.
Note that the unnamed module (where classpath classes live) is always considered to be `open`.

If this access check passes, the resultant `Lookup` will have full power access to everything that is accessible to the target class.

The disadvantage of this approach is that it does require the cooperation of module authors who would have to explicitly `open` the required packages.

## Next...

In the next post in this series, I will cover some more intermediate-level cases and talk a bit about proper API design around method handles.
