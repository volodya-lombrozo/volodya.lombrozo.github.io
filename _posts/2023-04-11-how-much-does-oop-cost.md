---
layout: post
title: "How much does OOP cost?"
date: 2023-02-06
---

In my work I quite often use objects because I'm a big fun an OOP development
because it helps me to implement and maintain the code much more effectively.

However, I'm sure, that you have heard a quite common phrase:
> OOP is a great idea, but it's not effective.

While it is a common used phrase, I haven't seen any real evidence that OOP
creates a significant performance overhead in most real applications.
So I decided to do some research here and just check how much does really OOP
cost.

All examples in that blog post are written in Java, but I'm sure that the
results will be more or less the same for other OOP languages like C++, C#,
Python, etc. Also important to mention the environment where I'm running the
tests, here is it:

```
Java version: Azul Zulu JDK 11.50.19 (11.0.12), 64-bit
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "12.5", arch: "x86_64", family: "mac"
```

For instrumentation profiling I'm using [VisualVM](https://visualvm.github.io)
and [async-profiler](https://github.com/async-profiler/async-profiler) as a
sampling profiler in rare cases.

## Plain example

As a first step I decided to check a simple example that shows the difference
between approaches and how much does it cost to use OOP comparing with the 
simple procedural approach.

Let's imagine we have the next class implemented the simple procedural approach
with static methods:
```java
class ExampleProcedure {
    private static int prime(int u) {
        return u;
    }
    private static int discounted(int u) {
        return prime(u) / 2;
    }
    public static void main(String... args) throws InterruptedException {
        int sum = 0;
        for (int i = 0; i < 2_000_000L; ++i) {
            sum += Example.discounted(i);
        }
        System.out.printf("Total: %d\n", sum);
    }
}
```
and OOP approach with Decorator pattern:
```java
class ExampleOOP {
    public static void main(String... args) throws InterruptedException {
        Thread.sleep(1000);
        int sum = 0;
        for (int i = 0; i < 2_000_000L; ++i) {
            Book b = new Discounted(new Prime(i));
            sum += b.price();
        }
        System.out.printf("Total: %d\n", sum);
    }

    interface Book {
        int price();
    }

    static class Discounted implements Book {
        private Book book;
        Discounted(Book b) {
            book = b;
        }
        @Override
        public int price() {
            return book.price() / 2;
        }
    }

    static class Prime implements Book {
        private int usd;
        Prime(int u) {
            usd = u;
        }
        @Override
        public int price() {
            return usd;
        }
    }
}
```
This examples shows the next results produced by VisualVM:
1. ExampleProcedure: 
2. ExampleOOP: Allocated objects 4_000_000, Allocated bytes: 64_000_000, Uptime: 9 sec
