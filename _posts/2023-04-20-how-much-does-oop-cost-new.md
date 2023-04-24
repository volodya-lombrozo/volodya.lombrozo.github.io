---
layout: post
title: "How much does OOP cost?"
date: 2023-04-20
---

# How much does OOP cost?

Have you ever wondered how much the object-oriented approach costs in terms of
the performance of your system? I'm sure you have. Despite discussions that
suggest modern compilers can solve many problems ahead of time, it's well-known,
and maybe even obvious, that OOP is still more expensive than the procedural
approach in terms of both CPU usage and memory consumption. In this blog post, I
will try to answer the question by comparing the two
approaches. The main purpose is to gain a high-level understanding of the
problem, or to check if the problem even exists.

This is only the first part of an overall investigation, which will concentrate
on the CPU performance penalty produced by the Object-Oriented Approach in Java
specifically. However, the results achieved could be applied to any other
object-oriented programming language.

# Related discussions

It can be challenging to find reliable information or discussions regarding the
topic at hand. To illustrate this point, let us consider two threads where
different conclusions were drawn. In
one [thread](https://stackoverflow.com/questions/372511/what-is-the-object-oriented-programming-computing-overhead-cost),
it was determined that the overhead of OOP is low, and most modern compilers can
address the issues associated with that approach. In contrast,
the participants in
another [thread](https://stackoverflow.com/questions/894465/will-changing-all-code-to-object-oriented-make-memory-usage-bigger-or-smaller)
opined that program performance is dependent on the quality and architecture of
the system code, rather than the programming approach. This implies that a
programmer can create a fast program using OOP and a wasteful program using a
procedural approach, which is a reasonable argument. However, int the same
thread one of the authors noted that OOP programs typically have higher memory
consumption, although no specific numbers were given as it was likely an
intuitive answer which, actually, was confirmed by
the [article](https://www.researchgate.net/publication/27382717_Evaluating_Performance_and_Power_of_Object-Oriented_Vs_Procedural_Programming_in_Embedded_Processors)
confirmed this observation, as it compared C++ and C languages to show a
significant penalty in OOP style programs compared to procedural style programs.
The study revealed that both execution time and memory consumption were
affected (CPU Penalty - 9.9% and Memory penalty 9.6% - 41%.)

In summary, while it may be difficult to find reliable information on this
topic, it appears that OOP programs tend to have higher memory consumption, and
there is evidence that OOP-style programs can have performance penalties when
compared to procedural-style programs. However, is this true in the Java world
or any other OOP language? Let's check it using two simple examples.

# Examples

The upcoming code samples provided in the blog will modify all two cases
discussed in that section to some extent. These code examples serve as a basic
introduction to procedural and OOP approaches. The
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
approach, specifically using
the [decorator](https://en.wikipedia.org/wiki/Decorator_pattern) pattern:

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

In all examples, I will mainly use [YourKit](https://www.yourkit.com) Java
Profiler in tracing mode with the instrumentation of Java bytecode. This
approach can significantly slow down the entire execution time, but it will
allow us to see the exact number of invocations and enable us to compare the two
approaches relatively. In rare cases, I may also
use [async-profiler](https://github.com/async-profiler/async-profiler) as a
sampling profiler and [JMH](https://github.com/openjdk/jmh) for
micro-benchmarking when it is necessary to clarify some behaviors or to run
examples without significant profiling overhead.
It is important to mention that when we say _execution time_, we are referring
to the [CPU time](https://en.wikipedia.org/wiki/CPU_time), which is actually
more accurate comparing with
simple [wall time](https://en.wikipedia.org/wiki/Elapsed_real_time).

# Overview

Okay, let's finally run both of our examples and see their execution times. We
will run all examples with all optimizations enabled and with the garbage
collector working, as in any other real-world scenario. Here are the
results:

| Approach      | Profiling tool    | Time (ms) |
|---------------|-------------------|-----------|
| 1. OOP        | YourKit (tracing) |           |
| 2. Procedural | YourKit (tracing) |           |
| 3. OOP        | JMH benchmark     |           |
| 4. Procedural | JMH benchmark     |           |

Interestingly, we found that both tools provided almost identical results,
showing that the OOP approach was around four times slower than the plain
procedural approach. The first idea that comes to mind when seeing this
difference is garbage collection (GC). In the procedural approach GC is not
utilized which could explain the faster execution time. However, we need to put
this theory to the test to confirm it.

# Garbage collection

The simples way to check the impact of GC is to disable it!

# Side effects

# Conclusion





