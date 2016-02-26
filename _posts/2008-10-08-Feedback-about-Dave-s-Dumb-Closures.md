---
layout: post
title: Feedback about Dave's Dumb Closures
categories: 
date: 2008-10-08 07:04:00
---
 I've gotten some feedback about [my closure idea](http://dmlloyd.blogspot.com/2008/09/daves-dumb-closures.html "") . Some of it bad, some of it... also bad. I thought I'd reply to all the comments in a new posting here rather than try to squeeze it all in to the crappy comment thing.

> Rémi Forax said...  
> You can't get ride of exception transparency, at least if you want to keep checked exceptions.  
> Here is an interresting email from Neal Gafter:  
> http://mail.openjdk.java.net/pipermail/closures-dev/2008-July/000174.html  
> Do you have a way to write StringSwitch with your syntax/semantics ?  
Rémi raises two separate points here, so I'll address them separately. Firstly, there's the question of "exception transparency". In response to this, please consider the following. First, in my proposal, the passed in code block may only be directly executed from within the receiving method. Therefore, the set of exceptions that may be thrown can be statically determined. The JVM can thus "safely" throw undeclared checked exceptions from the closure body as long as the calling method can handle it. The JVM would be able to verify this statically during bytecode validation.

Secondly, the question is whether one could use closures to write a StringSwitch as shown in Neal Gafter's email above. The short answer is: no. It is not possible to use closures in this way in Java. The reason is that Java has a linear stack; therefore a closure in Java may not outlive its lexical scope.

Now I know you're thinking, "Pfft, this is proof that BGGA closures are better". However this is where I point out that BGGA closures aren't closures, not in a JVM sense. They're just a syntax shortcut for anonymous inner classes. And the only way the so-called StringSwitch could possibly be implemented is basically as a syntactical wrapper around `Map<String,Runnable>` , with some magic to produce anonymous inner classes. This is not any different from what we have today; except possibly that today there is a rough correlation between amount of code written and efficiency of execution. With BGGA that all changes... it almost becomes easier to write bad code than good code.

> Yardena said...  
> Hi,  
> In languages that have closures, they are often used for "delayed evaluation", wrapping a block of code in a closure and passing on to some other object makes the code execute when the other object needs it and not right now. So closure can be used as an event handler, command, etc.  
> Unless I missed something major in your proposal, it does not support such a usage, since you don't support references to closures (I think this is what Remi is implying too).  
> So to get the record straight - you propose something much less powerful than BGGA, and not equivalent to closures in other languages.  
You can still have delayed execution; however the delayed execution must occur within the lifetime of the called method. This is, as I mentioned, a consequence of how lexical scoping works in Java. In languages that support "keeping" the closure past its declaration, the lexical scope is either not defined by the stack, or the stack is "forked" so that stack frames can be preserved even after they're out of scope. Java's stack is linear, and lexical scope is defined by the stack, so if you want closures, you're stuck with that. Of course BGGA does not have this limitation because BGGA isn't really closures. It's an abbreviated syntax over anonymous inner classes.

This all isn't to say that this problem is without solution. One could propose a JVM change to add support for forked stacks, for example. But I for one think that interfaces already solve this problem much more idiomatically (and honestly). Idiomatic because it's done that way today, and there's really nothing seriously wrong with the solution. Honest because you get what you pay for - you're saying you want to create an object, and that's what the JVM does - unlike BGGA in which you say you want to pass in a closure, and it goes ahead and creates an object anyway.

> Ricky Clarkson said...  
> "Basically this translates to Java as two things: A lexically-scoped block of code, and a means to execute that code or pass it on to another method for delayed execution."  
> Your proposal seems not to fit that second part, the delayed execution.  
> Suppose we're using a closurific wrapper around SwingUtilities.invokeLater:  
> doLaterClosurely() {  
> . . some gui stuff;  
> }  
> How would your proposal work for this?  
> It would be handy if you suggested what Java 5 code your examples translate to.  
Since I've answered the former question already, I'll jump right to the end: There **is** no Java 5 translation for my closure proposal. This is the whole point. Today the JVM does not have closures; after BGGA, the JVM still would not have closures. With my proposal, the JVM gets closures - real closures that really close over the actual lexical scope. This means support on a JVM level. My proposal simply assumes the minimum amount of change to make this so, which is limited to not adding support for forked stacks (which, by the way, would imply the need to deal with multiple threads accessing a single stack frame).

Again I need to emphasize that BGGA is moving away from the notion that what you write is what you get. Much like autoboxing, a mistake we continue to have to live with to this day, BGGA results in objects being automatically created on your behalf without your direct intention to do so. One end result of this is that brevity of code has a greatly reduced correlation to efficiency of bytecode or efficiency of execution. This is a mistake in my opinion. One reason that Java is considered to be so fast now is that the virtual machine maps very cleanly to a real machine, and the Java language maps cleanly to the virtual machine. So things that are easy to do in Java, by and large, can execute quickly on a real machine. Programmers who come from a C background, like myself, tend to find that things function and perform "as expected" on a real machine.

The advent of BGGA represents a departure in correlating language features with JVM capabilities, and thus the language no longer maps cleanly to a real machine. The natural side-effect of this is that you have to perform a lot of trickery (such as implicitly converting local variables to fields) to make it work. It is an undeniable fact that field accesses are slower than local variable accesses. It is an undeniable fact that a direct subroutine invocation (such as what you'd get with the JVM "jsr" instruction) is much faster than a virtual method lookup and invocation. Programs using BGGA closures are going to appear to be "slower" than programs that don't, because they will use more anonymous inner classes, and they will suffer from the implicit local variable-to-field conversion and virtual method invocation. I guarantee that this will be the case.

> WarpedJavaGuy said...  
> Why are new language constructs so often perceived as mutilations? Is changing the semantics of existing keywords and constructs not a mutilation?  
I actually have a fairly simple set of criteria here for Java. I have no problem with constructs that map to the virtual machine in simple, well-defined ways. Today, a local variable in Java means a local variable in the JVM. Even so-called "hacks" such as inner classes have a fairly clean JVM mapping when it comes down to it - they're still classes after all. Anonymous inner classes push things with their implicit copying of local variables into final fields. BGGA is over the line. With BGGA, you could measurably impede the performance of a method body (not to mention completely change 100% of the method bytecode) simply by putting a closure inside of it somewhere which accesses those local variables. **This** is what I call a "mutilation". You don't get what you pay for when you use BGGA closures.

All my proposal would require is either a way to define code blocks as an attribute of the calling method, with a way to correlate the lexical scope, or a way to jsr into other methods with parameters (see the "Nuts & Bolts" section of the original posting). This would make my closures a very simple mapping to JVM bytecode.

Thanks everyone for your questions - please feel free to send in more.