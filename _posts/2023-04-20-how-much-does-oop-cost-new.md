---
layout: post
title: "How much does OOP cost?"
date: 2023-04-20
---

# How much does OOP cost?

Have you ever asked that question? I'm sure you did. It's a well-known that
OOP cost more than the procedural approach from both sides - CPU usage and
memory consumption. When memory consumption is well described for many OOP
languages like for
[Python](https://stackoverflow.com/questions/372511/what-is-the-object-oriented-programming-computing-overhead-cost).
and we even have some works that tend to fix problems created by abuse OOP
paradigm.
When most of the publications and articles about OOP cost are focused on
...

# Examples

The upcoming code samples provided in the blog will modify all two cases
discussed in that section to some extent. These code examples serve as a basic
introduction to procedural and object-oriented programming (OOP) approaches. The
procedural approach will be implemented using a simple class that contains
static methods as follows:

```java
class ExampleProcedure {
    private static int prime(int u) {
        return u;
    }

    private static int discounted(int u) {
        return prime(u) / 2;
    }

    public static void main(String... args) {
        int sum = 0;
        for (int i = 0; i < 40_000_000L; ++i) {
            sum += Example.discounted(i);
        }
        System.out.printf("Total: %d\n", sum);
    }
}
```

As you can see, this is a simple procedural approach that uses static methods.
In contrast, the next example will achieve the same result using an OOP
approach, specifically using the decorator pattern:

```java
class ExampleOOP {
    public static void main(String... args) {
        int sum = 0;
        for (int i = 0; i < 40_000_000L; ++i) {
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

From now on, I will refer to these two code examples as the Procedural example
for the procedural `ExampleProcedure` class and the OOP example for
the `ExampleOOP` class. It's also worth mentioning that the total number of
iterations can be adjusted from the default value of `40,000,000` to make
certain differences more visible in some cases. However, any changes made to
this number will be clearly indicated.

# Hardware

Measuring the performance cost of OOP on different platforms can be difficult,
so for the sake of simplicity, we will be considering all examples in this blog
post on the same hardware and software stack. Although this may not be entirely
precise, it will give us a rough idea of what's going on and allow us to compare
the results with each other. Here is the stack we will be using:
```shell
CPU: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
RAM: 16 GB
OS: "mac os x", version: "12.5", arch: "x86_64", family: "mac"
Java: Azul Zulu JDK 11.50.19 (11.0.12), 64-bit
```
A diligent and curious reader can reproduce all the experiments using the same
hardware and expect to obtain the same results. Running the examples on
different platforms should also produce relatively similar results.

# Profiling tools

In all examples I will mostly use YourKit java profiling in tracing mode
(instrumentation of java bytecode) and async-profiler as a sampling profiler in
rare cases. Also I will use JMH for micro-benchmarking when it needed.
It's worth menthion that different profiling tools will give different time
numbers since they use different approaches to measure the time and have their
own overhead. For example, async-profiler.
All time measured will be in milliseconds and only CPU time is considered.

# Cases

# Overview

# Garbage collection

# Side effects

# Conclusion





