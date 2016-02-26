---
layout: post
title: Proper resource management
categories: 
date: 2008-07-28 21:22:00
---
 One of the most common problems that I see users having on the ##java IRC channel is improper resource management. By "resource management", I mean "things that need cleaning up after", such as streams, files, sockets, JDBC statements, etc. The tendency is not so much to ignore the problem, but to use an improper, unsafe, or just unnecessarily complex or ugly approach to solving it.

The problem is basically this. Suppose you open one or more resources, and some intermediate operation the middle of the code block fails. An exception is thrown, and without proper resource management, none of your files/sockets/database statements end up getting closed. This causes the reference to hang around until at least the next GC cycle, which can cause secondary problems like file descriptor exhaustion that are tough to diagnose.

To solve this problem, you don't need fancy closures or automatic resource management blocks. Java has a simple, rock\-solid way to completely avoid this issue already built into the language: finally blocks. Proper usage of this mechanism can produce code that is very safe and just as readable and elegant as the more recent "trendy" language proposals.

Consider this (incorrect) example which reads a couple lines from a file:

    // Example of INCORRECT code - DO NOT do this  
    try {  
        InputStream is = new FileInputStream("file.txt");  
        Reader r = new InputStreamReader(is);  
        BufferedReader br = new BufferedReader(r);  
        System.out.println("The first line is " + br.readLine());  
        System.out.println("The second line is " + br.readLine());  
        br.close();  
        r.close();  
        is.close();  
    } catch (IOException ex) {  
        // it failed :(  
        ex.printStackTrace();  
    }

It works perfectly as long as nothing fails. Of course if anything at all throws an exception in that block, then you may leak one or more resources (since the `close()` methods may not all be reached). There are many naive solutions to the problem, like this one that illustrates a couple different common errors:

    // INCORRECT solution - DO NOT do this  
    InputStream is = null;  
    Reader r = null;  
    BufferedReader br = null;  
    try {  
        is = new FileInputStream("file.txt");  
        r = new InputStreamReader(is);  
        br = new BufferedReader(r);  
        System.out.println("The first line is " + br.readLine());  
        System.out.println("The second line is " + br.readLine());  
    } finally {  
        is.close();  
        r.close();  
        br.close();  
    }

The first problem is simply that the values of `is`, `r`, and `br` are not checked for `null`. The solution here could simply be to check for `null` before calling `close`.

The second problem is that any of those three `close` statements can also throw an exception; this causes two additional problems: throwing an exception from a finally block will obscure any earlier exception, and if an earlier `close` fails, the later one will never be called. Solve this one by giving each `close` its own little `try/catch` block.

And the third (and maybe most subtle) problem is that even if you solved the prior two issues you miss one important thing. With many resources, you cannot consider an operation to be a success unless the `close` is also successful. And in general it is a good idea to always assume this is the case, and take advantage of the idempotency of the `close` operation (in other words, you can safely call it more than once without any additional effects beyond the first call). The solution here is to put a call to `close` for each resource inside the `try` block as well, so that exceptions are propagated but resources are also cleaned up.

So implementing these suggestions might leave us with something like this monstrosity:

    // Safe but very ugly example - I recommend against it  
    InputStream is = null;  
    Reader r = null;  
    BufferedReader br = null;  
    try {  
        is = new FileInputStream("file.txt");  
        r = new InputStreamReader(is);  
        br = new BufferedReader(r);  
        System.out.println("The first line is " + br.readLine());  
        System.out.println("The second line is " + br.readLine());  
        br.close();  
        r.close();  
        is.close();  
    } catch (IOException e) {  
        e.printStackTrace();  
    } finally {  
        if (is != null) try {  
            is.close();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        if (r != null) try {  
            r.close();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        if (br != null) try {  
            br.close();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }

That's 30 lines of code to read and print two lines from a file \- yuck! There is in fact a more elegant way to solve this problem. The first step is to restructure the code with separate `try/finally` blocks for each resource:

    // Safe, but really just as ugly...  
    try {  
        final InputStream is = new FileInputStream("file.txt");  
        try {  
            final Reader r = new InputStreamReader(is);  
            try {  
                final BufferedReader br = new BufferedReader(r);  
                try {  
                    System.out.println("The first line is " + br.readLine());  
                    System.out.println("The second line is " + br.readLine());  
                    br.close();  
                    r.close();  
                    is.close();  
                } finally {  
                    try {  
                        br.close();  
                    } catch (IOException e) {  
                        e.printStackTrace();  
                    }  
                }  
            } finally {  
                try {  
                    r.close();  
                } catch (IOException e) {  
                    e.printStackTrace();  
                }  
            }  
        } finally {  
            try {  
                is.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
    } catch (IOException e) {  
        // failure  
        e.printStackTrace();  
    }

Notice two things \- first, I've switched to `final` variables to hold the resource. This really underlines the point that we do not need a `null` check anymore for each resource. That's one problem solved. Second, there's more of a structure to the resource management that emphasizes their lexical scoping. But look how huge it is \- we're actually doing worse by about 7 lines!

The final enhancement is the addition of a static method to safely clean up a resource. It looks like this:

    // put this anywhere you like in your common code.  
    public static void safeClose(Closeable c) {  
        try {  
            c.close();  
        } catch (Throwable t) {  
            // Resource close failed!  There's only one thing we can do:  
            // Log the exception using your favorite logging framework  
            t.printStackTrace();  
        }  
    }

Now our code can be changed to look like this:

    // Now this is more like it - readable and foolproof!  
    try {  
        final InputStream is = new FileInputStream("file.txt");  
        try {  
            final Reader r = new InputStreamReader(is);  
            try {  
                final BufferedReader br = new BufferedReader(r);  
                try {  
                    System.out.println("The first line is " + br.readLine());  
                    System.out.println("The second line is " + br.readLine());  
                    br.close();  
                    r.close();  
                    is.close();  
                } finally {  
                    safeClose(br);  
                }  
            } finally {  
                safeClose(r);  
            }  
        } finally {  
            safeClose(is);  
        }  
    } catch (IOException e) {  
        // failure  
        e.printStackTrace();  
    }

All of the original problems are solved, and in addition it is a lot easier to see what is going on in terms of resource cleanup. This is by no means the only solution to the problem, and certainly not the shortest, but I believe it is the cleanest and safest (in terms of human error).

The same kind of thing can be applied to JDBC handles (just write a `safeClose()` method for connections and statements) or just about anything else that needs cleaning up after.