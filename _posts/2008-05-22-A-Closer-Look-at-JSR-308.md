---
layout: post
title: A Closer Look at JSR-308
categories: 
date: 2008-05-22 19:58:00
---
 If you haven't heard of JSR-308, take a few minutes to [read the specification]("http://groups.csail.mit.edu/pag/jsr308/java-annotation-design.html" "") and familiarize yourself with it. This JSR is designed "permit annotations to appear in more places than Java 6 permits".

A number of blogs have popped up with negative reactions to this JSR, like [this one]("http://www.michaelnygard.com/blog/2008/05/when_should_you_jump_jsr_308_t.html" "") and [this one]("http://bc-squared.blogspot.com/2008/05/what-hath-java-wrought.html" "") , recently [featured on TSS]("http://www.theserverside.com/blogs/thread.tss?thread_id=49416" "") . While I agree with the given opinions in many ways, it is unfortunate that there is not a great deal of technical depth or logical reasoning given in these posts (mostly it's statements to the effect of "makes Java too complex"), so I'd like to take this opportunity to explain in detail exactly why this JSR, as it stands right now, is bad for Java and its community, by citing specific examples.

I think the best place to start is to look at [section B.2 of the working draft]("http://groups.csail.mit.edu/pag/jsr308/java-annotation-design.html#htoc16" "") of JSR-308, which is the section that outlines the proposed uses for annotations on types. The justification for any change to Java (or to any language really) should be rooted in a valid use case, and many use cases are given in this section. Here's why few, if any, of the given use cases are valid justification for the changes outlined in this JSR.

###    Generics and arrays

There are multiple use cases given for applying type annotations to arrays and generic collections. Here's the very first one given:

> The Titanium ... dialect of Java requires the ability to place the local annotation (indicating that a memory reference in a parallel system refers to data on the same processor) on various levels of an array, not just at the top level.

Fine, fine. Titanium sounds lovely. But, we're talking about Java here. You are implying that there is a need for such an annotation within Java itself. But how would Java ever be able to effectively make use of such an annotation, without substantial changes to the compiler and/or VM? Java solves concurrent problems quite well as it stands right now, especially since the introduction of the "java.util.concurrent" package in JDK5, and there is a very active sub-community, populated with some very smart people, focused around further improving concurrency in Java by expanding upon the proven model that exists now.

It's fine to explore other concurrency models, but in my opinion it would be a disservice to the Java community to do this in Java itself. How about providing a special-purpose language on top of the JVM for this? I've heard good things about Scala, for example.

There is **no** need or justification to re-solve a problem that has already been solved in a way that has been proven to be very effective. Therefore I believe this use case to be wholly invalid.

> In a dependent type system [...], one wishes to specify the dimensions of an array type, such as Object[@Length(3)][@Length(10)] for a 3×10 array.

Java does not have a dependent type system as described here. So... how is this a valid use case? Are you proposing that Java's type system be changed? Are you sure you want to open that can of worms? What problem are you trying to solve here? The traditional way to enforce strict bounds on an array as proposed here is to wrap the array manipulation operations in a class. Compared to using annotations, using a wrapper class is potentially much easier to read, and I for one would certainly rather explain arrays as they exist today, rather than explain arrays plus all the annotations you can stick to them, and what they mean, to a clueless newbie.

> An immutability type system [...] needs to be able to specify which levels of an array may be modified. Consider specifying a procedure that inverts a matrix in place.

See my previous point, because it applies equally well here. An array is not a matrix. I know lots of traditional computer scientists would love it if they were, but they simply are not - if you want a matrix, make a Matrix class and enforce your policy there. This is a solution that has served us well until now; there is no reason to reinvent this, especially at this late stage.

> An ownership domain system [...] uses array annotations to indicate properties of array parameters, similarly to type parameters.

I took a few minutes and read through the paper referenced by this point, and I found that they demonstrate an implementation that is compatible with Java 5 annotations as they stand. The section (6.2) in which they expound upon the limitations of Java 5 annotations that they ran into, does not seem to mention any deficiencies that are solved by this JSR. So this one is a bit of a question mark for me.

> The ability to specify the nullness of the array and its elements separately is so important that JML [...] includes special syntax \nonnullelements(a) for an array a with non-null elements.  
> In a type system for preventing null pointer errors, using a default of non-null, and explicitly annotating references that may be null, results in the fewest annotations and least user burden [...]. Array elements can often be null (both due to initialization, and for other reasons), necessitating annotations on them.

This is referring to [JML]("http://www.cs.ucf.edu/~leavens/JML/" "") and the principle of design by contract. And many will agree (myself included) that it is an interesting idea - you statically define your contract in great detail along side your method declarations or definitions, and users of that interface can perform extensive validation to verify that they conform to that contract at compile time.

But: does DBC belong in Java itself? In my opinion, no: it represents a fundamental change to the way that APIs are specified in Java. If JML-style constraints were introduced in to Java itself, you'd be taking away the language we know as Java, and replacing it with a new language. While I'm not against design by contract by any stretch of the imagination, I think that a DBC-based language is best implemented as a new language altogether.

That said, the specific use case given by these two points - a null and/or non-null annotation - is something that can be quite useful, and in fact can be comprehensively implemented using the annotation system that has existed since JDK5. In fact I'd love to see it - but the (major) caveat is that in order to be truly useful, pretty much the entire JDK would have to be retroactively annotated. There is a bit more to be said about this type of annotation in a moment.

I do NOT believe that it is worthwhile to extend this type of annotation to arrays. Arrays have always been and will always be a special case in Java. In my opinion, the rules for arrays are weird/different enough from a "regular" object that they're best left as is - they solve a relatively narrow problem domain quite well enough. Expanding the scope of arrays introduces more risk and/or complexity than benefit (again, my opinion).

###    Receivers

This is a weird one. The basic premise is that on a (non-static) method, in addition to your regular method parameters, you should be able to put an annotation on "this", because it's kinda-sorta a formal parameter. Sorta.

The problem is that no, it's not a formal parameter at all. You can't, for example, modify the actual "this" reference in your method. In fact the one single example that is given is a "@Readonly" annotation that applies to the method receiver, which means that the given method doesn't modify its receiver. This is of course ludicrous, because you can't modify "this". You can certainly dereference it, which is I guess (?) what they're getting at. However there is no reason why this is cannot a method-level annotation which basically means "this method does not mutate the object state".

By the way, on more than one occasion, the JSR implies that annotations on a method are associated with the method's return value. While true in some cases, this is absolutely untrue in general. Many, many annotation-based tools and frameworks use method-level annotations to indicate the method itself. And very seldom (in fact, never in my experience) is there confusion about whether an annotation applies to the method or the method's return value.

So I don't consider this use case valid at all.

###    Casts

This section, especially in combination with the null/not-null example, very much demonstrates a couple of key issues I have with this JSR.

The thing that really gets me about the whole nullability example is that this is a concept that does not even apply to types. Nope, not even a little. Nullability is relevant to **variables** (local variables, fields, parameters) and **values** (null is obviously always null; constants and many other types of expressions, like object construction, are safe to call non-null); but not types. It is a constraint on the value space of a variable which is completely orthogonal to the type system.

Let me give you an example. Pretend that we've got an annotation called "@NotNull", which can be applied to local variables, fields, methods (referring to the return value of the method), and method parameters. This is a horribly contrived snippet of a method that uses the hypothetical "@NotNull" annotation in various ways:

      ...  
      Object bar = somePlainOldMethod();  
      // Permissible because the JVM can prove that foo is not null  
      @NotNull Object foo = new Object();  
      // At this point, foo (the variable) may NOT contain null  
      // So, this would be a compile-time error  
      foo = null;  
      // As would this (bar might be null)  
      foo = bar;  
      // Now it gets interesting:  
      if (bar != null) {  
          // Hmm, the compiler can easily prove that bar cannot be null here!  
          // That makes this line a-ok - and without a big dumb (@NonNull Object) cast!  
          foo = bar;  
      }

Notice the inference in the "if" statement. Wouldn't it be nice if Java were always that smart? Do you see now how the nullness of a variable has no relation to its type? The variable "bar" was nullable, then provably non-null, then nullable again. The JSR adds this kind of inference as merely an optional footnote, preferring that you use the cast syntax instead.

Oh, and note that all this can be done with annotations as they exist today.

###    Type tests

OK, this one is even stranger. The idea is that an "instanceof" test becomes dual purpose. On the one hand, it's a runtime check of variable type. But it also becomes a compile-time annoyance generation mechanism. How so? It basically adds a useless constraint - you must specify all your local variable annotations in your "instanceof", or get a compile error. If you do so, you're rewarded with - wait for it - **zero** additional functionality; you're right where you started.

Plus, this opens the door to newbie programming errors like this (quoted directly from the actual working draft):

    if (x instanceof @A1 T) { ... }  
    else if (x instanceof @A2 T) { ... }

Think about what that means. The second "if" is never true, because the annotation is evaluated at compile-time, while the "instanceof" is evaluated at runtime!

This change is not only useless and annoying, but it seems to bear little or no relationship to the other proposed use cases. It's a solution looking for a problem.

###    Summary

There are a few other smaller sections but there are really no other concrete examples worth commenting additionally on, so I'll leave the rest as an exercise for the reader.

This is how I see it. This JSR will add a much higher level of complexity to a language that has already greatly increased in complexity in the past few years. You are changing a language that many people use to a language that, in my opinion, many fewer people will use, because of the increased learning curve and large increase in complexity for what I see as very minimal gain. Many (including myself) feel that this JSR is crossing a threshold, beyond the so-called point of diminishing returns.

Many of the problems which this JSR intends to solve are important and relevant to those who have identified them. But you can't (and shouldn't) solve every problem in Java, and it is my belief that you will find that most of the problems which this JSR was introduced to solve are not important to most Java users.

One of Java's strengths is that it is a language that compiles to a portable bytecode. How about taking some of these ideas and applying them to new languages that target the JVM?