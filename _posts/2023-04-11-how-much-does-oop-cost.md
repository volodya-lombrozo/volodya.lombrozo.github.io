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

The results produced by JMH (with warmup and many invocations):

1. ExampleProcedure: Allocated objects: 0, Allocated bytes: 0, Average execution
   time: 1.127 ms
2. ExampleOOP: Allocated objects 4_000_000, Allocated bytes: 64_000_000,
   Average execution time: 8.025 ms

JMH configuration is common for all tests:

```java
new Runner(
    new OptionsBuilder()
    .forks(1)
    .warmupIterations(1)
    .measurementIterations(1)
    .warmupTime(TimeValue.seconds(3))
    .measurementTime(TimeValue.seconds(3))
    .mode(Mode.AverageTime)
    .timeUnit(TimeUnit.MILLISECONDS)
    .build()
    ).run();
```

The results produced by calculating time manually using `System.nanoTime()`
command with GC:

1. ExampleProcedure: Allocated objects: 0, Allocated bytes: 0, Average execution
   time: 8 ms
2. ExampleOOP: Allocated objects 4_000_000, Allocated bytes: 64_000_000,
   Average execution time: 33 ms

As you can see, the OOP approach is much slower than the procedural one (~x8).
But why OOP approach cost so much?

## Garbage Collection

Let's try to disable garbadge collection and check the results again:

I've disabled GC by using EpsilonGC:

```
-Xmx12294M -Xms12294M -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC
```

The results produced by JMH (with warmup and many invocations):

1. ExampleProcedure: Allocated objects: 0, Allocated bytes: 0, Average execution
   time: 1.067 ms
2. ExampleOOP: Allocated objects 4_000_000, Allocated bytes: 64_000_000,
   Average execution time: 51.308 ms

The results produced by calculating time manually using `System.nanoTime()`
command with GC:

1. ExampleProcedure: Allocated objects: 0, Allocated bytes: 0, Average execution
   time: 6 ms
2. ExampleOOP: Allocated objects 4_000_000, Allocated bytes: 64_000_000,
   Average execution time: 42 ms

How does it possible? We don't spend time for GC, but the execution time is
increased. Apparently, allocating new objects in memory without garbage
collection can be challenging and inefficient. Disabling garbage collection can
cause your program to spend more time managing memory, which can slow down its
overall execution time. 