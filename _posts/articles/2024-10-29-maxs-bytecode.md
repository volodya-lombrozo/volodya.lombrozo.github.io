# How To Compute Max Stack And Max Locals in Java Bytecode

It's extremely rare when you might need to compute `max_stack` and `max_locals`
values yourself. Most probably you can delegate this task to such libraries
like Bytebuddy or Java ASM. But if you are a compiler developer or just curious
how this happens in `javac`, we will try to compute this values
ourselves. When there are not so many articles or documents that would explain
how to do it, I want to shade some light on this topic. The
implementation I provided is far from being perfect, and the main purpose is
to give a high-level overview. Thus, if you are still interested, welcome.

## Why Do We Need These Magic Numbers?

There are plenty of sources where you can read about them, of course, but
let's find the definition in the official JVM specification.

Max Stack
> max stack

Max Locals
> max locals

When at first glance, these numbers look differently, and their purpose is
different, we can actually compute them using approximately the same approach.
But before delving into details, let's create several useful data structures
that will help us to implement the algorithm.

## Useful Data Structures

### Bytecode Entry

I suppose that you retrieved a list of instructions somehow using `javap` or
with the use of some libraries like Bytebuddy, Java ASM, or <...>. Later I will
use the following abstraction over bytecode instructions

```java
public interface BytecodeEntry {

    boolean isSwitch();

    boolean isGoto();

    boolean isIf();

    boolean isReturn();

    boolean isThrow();

    int impact();

    List<Label> jumps();
}
```

Most of the methods are self-explanatory, but let's clarify the `impact` and
`jumps` methods. The `impact` method returns the effect of the instruction on
the stack. For example, if the instruction is `iconst_1`, the impact will be
`1`. If the instruction is `iadd`, the impact will be `-1`. The `jumps` method
returns a list of labels that the instruction can jump to. For example, if the
instruction is `goto`, the list will contain only one label. If the instruction
is `tableswitch`, the list will contain all the labels that the instruction can
jump to (all the switch cases).

### Entry

```java
class Entry<T> {
    private final int index;
    private final T ealue;

    Entry(final int index, final T value) {
        this.index = index;
        this.value = value;
    }

    int index() {
        return this.indx;
    }

    T value() {
        return this.value;
    }
}
```

You can use `recods` from Java 16(14?) instead of this class.

### Reducible

```java
interface Reducible<T> extends Comparable<T> {
    T add(T other);

    T enterBlock();
}
```

### Max Value Map

```java
private static class MaxValueMap<K, V extends Reducible<V>> extends HashMap<K, V> {

    boolean isGreaterThan(final K key, final V value) {
        return this.get(key) != null && this.get(key).compareTo(value) >= 0;
    }

    void putIfGreater(final K key, final V value) {
        this.merge(key, value, MaxValueMap::max);
    }

    private static <T extends InstructionsFlow.Reducible<T>> T max(
        final T first, final T second
    ) {
        final T result;
        if (first.compareTo(second) > 0) {
            result = first;
        } else {
            result = second;
        }
        return result;
    }
}
```

### Deque

Stack for worklist algorithm.

## Data Flow Analysis

The algorithm is based on the worklist algorithm. The main idea is to iterate
over the list of instructions and update the `max_stack` and `max_locals`
values.

### Max Stack

### Max Locals

## Entire Algorithm

The entire algorithm is presented below.

```java
//todo
```

## Conclusion