---
layout: post
title: Java Language Support for Tail Recursion
categories: 
date: 2008-09-30 10:23:00
---
Everyone wants tail recursion. I've heard that some JVMs automatically detect tail recursion even. How about a simple syntax change to do it at a language level?

        public int factorial(int number) {  
            if (number == 0) {  
                return 1;  
            } else {  
                goto factorial(number, 1);  
            }  
        }  
        private int factorial(int current, int sum) {  
            if (current == 1) {  
                return sum;  
            } else {  
                goto factorial(current - 1, sum * current);  
            }  
        }

Long live `goto` ! In this example, `goto` means "replace the current stack frame with a call to this method", where the given method can be any method whose return value is compatible with, or equivalent to, the calling method's return value.