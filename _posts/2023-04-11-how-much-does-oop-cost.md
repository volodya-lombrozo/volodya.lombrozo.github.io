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

I've asked
the [question](https://stackoverflow.com/questions/75996839/disabling-garbage-collection-in-java-unexpectedly-slows-performance)
on the StackOverflow and havn't got any answer yet.

Then I continued profiling by YourKit. In order to get more stable results
I've increased the number of iterations to `40_000_000` for all test examples.
It worth nothing that YourKit used tracing profiler that significantly increases
the overhead of the profiling itself. Thus, it's important to analyze the
results carefully. The results:

1. ExampleProcedure: Allocated objects: 0, Allocated bytes: 0, Average execution
   time: 9.792 ms
   ExampleProcedure.java:13 procedure.ExampleProcedure.discounted(int) 3885 ms
2. ExampleOOP (with GC): Allocated objects 40_000_000, Allocated bytes:
   640_000_000, Average execution time: 29.193 ms
   ExampleOOP.java:9 oop.ExampleOOP$Discounted.price() 3873 ms
   ExampleOOP.java:8 oop.ExampleOOP$Discounted.<init>(ExampleOOP$Book) 3856 ms
   ExampleOOP.java:8 oop.ExampleOOP$Prime.<init>(int) 3845 ms
3. ExampleOOP (without GC): Allocated objects 40_000_000, Allocated bytes:
   640_000_000, Average execution time: 28.347 ms
   ExampleOOP.java:9 oop.ExampleOOP$Discounted.price() 3719 ms
   ExampleOOP.java:8 oop.ExampleOOP$Discounted.<init>(ExampleOOP$Book) 3689 ms
   ExampleOOP.java:8 oop.ExampleOOP$Prime.<init>(int) 3675 ms

As you can see, the results are more or less the same between ExampleOOP with GC
and ExampleOOP without GC. It means that GC almost doesn't affect the execution
time. Also, it's important to note that YourKit profiler gives more realistic 
results comparing with JMH and doesn't show the problem with disabling GC.

The next problem which I see is that the OOP approach spends more time for the 
loop in the `main` method:
1. ExampleProcedure: 5907 ms
2. ExampleOOP (with GC): 17619 ms
3. ExampleOOP (without GC): 17264 ms

The possible reason in the dynamic dispatching. The JVM has to check the type
of the object and choose the appropriate method to execute.
