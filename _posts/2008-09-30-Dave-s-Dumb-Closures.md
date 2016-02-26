---
layout: post
title: Dave's Dumb Closures
categories: 
date: 2008-09-30 01:12:00
---


####  Or: How to add closures to Java without mutilating the language

Those of you know have spoken with me via any media for more than about 45 seconds consecutively know that I am vehemently opposed to the various closure implementations for Java that have been proposed for inclusion in to Java 7. I'm not opposed to closures, mind you \- I just really dislike the implementation options that are currently on the table. At one point I had visualized writing The Final Blog Post On The Subject, which would be so sharp of wit that any possible counterargument simply could not stand in the face of it. It would be eloquent, noble, and completely irrefutable \- even the staunchest BGGA supporter would be won over.

Then reality sets in, and one is reminded that pretty much every such argument against the different closure ideas has been made, and promptly deflected. So rather than add to that particular category of noise, I thought I'd present a rough outline of a closure implementation that would sit much better with me, in the hopes that there are folks out there who have the same misgivings as I do about the current state of things.

####  What is a Closure?

A closure is defined in <a href="http://foldoc.org/?closures">FOLDOC</a>as "a data structure that holds an expression and an environment of variable bindings in which that expression is to be evaluated."

Basically this translates to Java as two things: A lexically\-scoped block of code, and a means to execute that code or pass it on to another method for delayed execution.

The use cases for these things are numerous. Other imperative languages have used them for such things as customized control flow structures, continuations, resource management, and so on. Here in Javaland though, we are left out in the cold \- the closest thing we get to these fun toys are anonymous inner classes. And while anonymous inner classes give the impression of being a sort of limited closure (with the caveat that it can only access local variables that were declared final \- they're actually copied to fields transparently), they're obviously not the real deal.

The current leader, BGGA, takes the approach of taking the anonymous inner class notion a few steps farther \- local variables aren't just copied to fields, they **become** fields (and thus read/write) \- along with lots of language glue (magic dynamically\-generated types, lots of new syntax and rules, some special annotations) to make it all work sanely. I'm not going to go into an exhaustive list of What's Wrong With BGGA; rather I'm going to demonstrate a completely different notion of a closure in Java from what you've seen in BGGA and the other competing specs.

####  A Closure is a Procedure

My view of a closure in Java is simply an inner method \- or more accurately, an inner procedure. This differs from BGGA (and anonymous inner classes) in a few ways:

* Local variables are local variables. They're all on the stack; the stack is still the regular linear LIFO we're all used to. This means no surprises with respect to concurrent access or comparatively slow field accesses behind the scenes.

* No objects floating out on the heap represent the current lexical scope. No heap means no GC cost \- exiting the calling method instantly cleans up the scope.

* Exception handling is greatly simplified \- the exception types can be inferred from the inner block.

* Closures are (obviously) not inner classes anymore; they cannot be used outside of the lexical scope of the calling method.

* The whole function\-type interface hierarchy is gone. This means that calling your closure is no longer a virtual method dispatch. It becomes a simple pointer dereference. These facts come about because the implementation I describe creates **real closures** (not closures emulated with objects) over **real lexical scopes** (not lexical scopes that are really collections of fields in a synthetic object class).

Since many people are visual in nature, allow me to demonstrate a possible syntax and usage for these things.

This first example would be a typical ARM (automatic resource management) scenario. Imagine a Lock interface with a method like this:

        public interface Lock {  
            void hold() for theBlock();  
        }

Notice the use of the "for" keyword. Yeah, I stole that idea from BGGA. Anyway, here's how you'd implement it:

        public class MyLock implements Lock {  
            // ...stuff...  
            public void hold() for theBlock() {  
                ...acquire the lock using ninja CAS techniques...  
                try {  
                    theBlock();  
                } finally {  
                    ...release the lock unconditionally...  
                }  
            }  
        }

Finally you'd use it like this:

        private final Lock lock = new MyLock();  
        public void doStuff() throws TheException {  
            // we need to access the shared resource!  
            lock.hold() for() {  
                // ok, let's do some test  
                if (someConditionIsWrong) {  
                    throw new TheException("Failure...");  
                }  
            }

Or we could ditch the "for" on usage if there's no parameters:

        lock.hold() {  
            // do stuff  
        }

Niiiiice.

Now think about this for a moment. Not only does it *look* awesome, but we've removed the virtual method invocations you'd get with BGGA thereby making this potentially super\-fast, eliminated an entire *category* of new magi\-types from the language, and solved the Big Exception Conundrum.

Did I go too fast? Let me back up and cover those points.

First \- it looks awesome, you can't deny it. No goofy arrows pointing from nothing to whatever, no weird `({()})` crap, etc. It reads easily in declaration, definition, and invocation, and it's not a lot of typing.

Next \- consider what this closure structure does for performance. With one single method invocation, we are locking the resource, executing the user block, and unlocking the resource. You can't even beat that using try/finally today \- at best you've got a method call to lock and a method call to unlock, with your code in between (I'll get into the mechanics of how the call to the block actually occurs down below, for you skeptics out there).

I won't cover the magical auto\-created interface types in BGGA \- if you know what I'm talking about, well, then you know what I'm talking about. If not \- you don't want to know, believe me.

Finally, the Big Exception Conundrum. Due to the fact that the closure is really and truly lexically scoped \- not fakey\-lexical\-scoping like anonymous inner classes and their big BGGA brother \- all kinds of tricky issues about exception propagation simply disappear. The complete relevant part of the call stack is statically known at compile\-time! Therefore there's no need to put in special plumbing for the closure to have some fancy dynamic `throws` list, or for the method which invokes the closure to have any knowledge or regard for what the closure may throw. It will throw whatever the closure's parent method can catch or propagate at the point that the closure is defined, just like a regular code block. And nonlocal returns can be trivially implemented using the same techniques.

####  Some More Lovely Examples

The oft\-wished\-for Map pairs iteration \- interface:

        interface Map<K, V> {  
            // ...  
            void iterate() for block(K key, V value);  
            // ...  
        }

Implementation:

        public class FooMap<K, V> implements Map<K, V> {  
            // ...  
            // cheapo implementation just uses good old entrySet()  
            public void iterate() for block(K key, V value) {  
                Set<Entry<K, V>> entrySet = entrySet();  
                for (Entry<K, V> e : entrySet) {  
                    block(e.getKey(), e.getValue());  
                }  
            }  
            // ...  
        }

Usage:

        Map<String, Thing> things = new FooMap<String, Thing>();  
        // ...later...  
        things.iterate() for (String key, Thing value) {  
            System.out.println("The string " + key + " is for the thing " + value);  
        }

Think how difficult it was for the BGGA crowd to get control structures in there! And they're still pretty ugly if you ask me (at least in v0.5 of the draft):

        // weird BGGA stuff...  
        <R, T extends java.io.Closeable, throws E>  
        R with(T t, {T=>R throws E} block) throws E {  
            try {  
                return block.invoke(t);  
            } finally {  
                try { t.close(); } catch (IOException ex) {}  
            }  
        }

This same safe\-close control structure would be done like this (notice how absolutely no type parameters are needed):

        public static void with(Closeable resource) for block() {  
            try {  
                block();  
            } finally {  
                try {  
                    resource.close();  
                } catch (Throwable t) {  
                    // log it, of course!  
                }  
            }  
        }

Then the usage would be:

        InputStream is;  
        with (is = new InputStream("foo.txt")) {  
            // read me my bytes!  
        }

Or how about slick user transaction management (usage only shown):

        txn.begin() {  
            // do some JDBC or Hibernate or something!  
        }  
        // or keep a threadlocal with the context and you could do:  
        transaction() {  
            // do some JDBC or Hibernate or something!  
        }

Or a nice way to handle streaming result sets from a file or database (usage only shown):

        data.fetch() for (String lastName, String firstName, BigDecimal balance) {  
            // Take a hike, data objects.  These are all local variables!  Talk about cheap GC!  
            System.out.println(lastName + ", " + firstName + " owes us $" + balance);  
        }

Privileged actions get a performance boost (no object creation, no virtual method dispatch) \- and say goodbye and good riddance to PrivilegedExceptionAction:

        AccessController.doPrivileged() {  
            secretStuff.activate();  
        }

####     Nuts & Bolts

The way I describe these closures is based on a few key technical ideas.

* The code block, when it executes, **maintains a "link" to the outer stack frame** . This is what allows the outer variables to be accessed. All local variables continue to live on the stack.

* As such, the actual internal value of the closure parameter is either a pair consisting of the parent stack frame address and the code block address, or a pointer to a **trampoline** \- a small bit of machine code, generated at runtime, which resides on the stack and initializes the child stack frame with said pointer before passing execution to the inner code block.

* A code block would probably have to have its own type designation, as a new primitive type (they're like methods, but without a return value and with no exception list).

* I assume that all returns are nonlocal, though it is not required by this design. Nonlocal returns still pass through all the `finally` blocks that would be on the call stack.

* A code block may implicitly throw any checked exception that is catchable at the point that the method (the one which accepts the closure) is implemented. This includes, of course, any checked exceptions declared on the (closure\-accepting) method itself.

* Variables may not be declared as a closure type. This helps keep the magic smoke inside the computer.

* Closures only live for the duration of their lexical scope. Their REAL lexical scope, not a pretend scope that's really an object.

* In fact, the implementation may not consist of an object to substitute for a lexical scope. Violators will get scowled at, and possibly called rude names.

* No fancy parser acrobatics are needed \- only the `for` epilogue to method declarations and the method\-call postfix parameters and code block are needed. No additional operators are introduced.

* The bytecode file format would be enhanced to include attribute sections for code blocks; these need only be indexed by number.

####  Other Ideas Based On Real Closures

* Inner methods! For all those pesky private/private\-static methods you have that only ever get called from one method. Inner methods can be referenced directly without looking them up by name, since they cannot be accessed outside of their lexical scope (not even through reflection).

* Very cheap threadlocal storage. The lexical scope is guaranteed to be locked to a single thread's stack. This could be taken advantage of... somehow. What other weird and wonderful stuff can you think of?