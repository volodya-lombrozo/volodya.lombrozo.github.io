---
layout: post
title: ""
date: 2025-02-19
---

Do you think if the method `children()` is thread safe?

```java
import java.util.stream.Stream;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;

public final class SafeXml {
    private final Node node;

    SafeXml(final Node node) {
        this.node = node.cloneNode(true);
    }

    public Stream<SafeXml> children() {
        NodeList nodes = this.node.getChildNodes();
        int length = nodes.getLength();
        return Stream.iterate(0, idx -> idx + 1)
            .limit(length)
            .map(nodes::item)
            .map(SafeXml::new);
    }
}
```

Of course, since I'm asking this question, the answer is "No". For those, who
didn't work with XML in Java (yes, it's still alive), the classes under
`org.w3c.dom` package are not thread safe. That is. There are no guarantees
even for reading the data from the XML document in a multi-threaded environment.

So, if you decide to run the `children()` method in parallel, you will get

```java
Caused by:java.lang.NullPointerException
    at java.xml/com.sun.org.apache.xerces.internal.dom.CoreDocumentImpl.getNodeListCache(CoreDocumentImpl.java:2283)
    at java.xml/com.sun.org.apache.xerces.internal.dom.ParentNode.nodeListGetLength(ParentNode.java:690)
    at java.xml/com.sun.org.apache.xerces.internal.dom.ParentNode.getLength(ParentNode.java:720)
    at com.github.lombrozo.xnav.SafeXml.children(SafeXml.java:41)
```

sooner or later.  (Java uses Xerces as the default implementation of the DOM
API.)

For those who are interested, here is a good stackoverflow question about this
https://stackoverflow.com/questions/3439485/java-and-xml-jaxp-what-about-caching-and-thread-safety

Let's fix it then!
Since we are experienced developers, we go to StackOverflow, and here
is [solution](https://stackoverflow.com/questions/10550900/concurrency-and-reusage-of-org-w3c-dom-node):

> your only choice is to synchronize all access to the Document/Nodes


So, let's add a bit of synchronization to the `children()` method:

```java
public Stream<SafeXml> children(){
synchronized (this.node){
    NodeList nodes=this.node.getChildNodes();
    int length=nodes.getLength();
    return Stream.iterate(0,idx->idx+1)
    .limit(length)
    .map(nodes::item)
    .map(SafeXml::new);
    }
    }
```

Good. Let's test it. Test makes many threads to call `children()` method in
parallel,
shows green passes. ... And failed on 269th run. The
same `NullPointerException`.
How that? We added synchronization, didn't we?

I will try to save your time here and tell you that the problem is that we are
synchronizing on the `Node` object, but the problem here is that all nodes have
some internal state which is not thread safe and shared between all the nodes.

Well, we have the only last resort here: we need to synchronize on the entire
document.

```java
public final class SafeXml {

    private final Document doc;
    private final Node node;

    SafeXml(final Document doc, final Node node) {
        this.doc = doc;
        this.node = node.cloneNode(true);
    }

    public Stream<SafeXml> children() {
        synchronized (this.doc) {
            NodeList nodes = this.node.getChildNodes();
            int length = nodes.getLength();
            return Stream.iterate(0, idx -> idx + 1)
                .limit(length)
                .map(nodes::item)
                .map(n -> new SafeXml(this.doc, n));
        }
    }
}
```

We might not like it, but it's the only way to make it work. And this behavior
is specifically and historically was designed this way in the DOM API.
If developers make a decision to make it thread safe, it would be a huge
performance hit for the single-threaded applications. So, it's a trade-off.
We might not like it, but we have to live with it.

After this fix I ran the test again and it passed all the time. So, I decided
to make a cup of coffee. When I returned, I've saw this:

```java
Caused by:java.lang.NullPointerException
    at java.xml/com.sun.org.apache.xerces.internal.dom.CoreDocumentImpl.getNodeListCache(CoreDocumentImpl.java:2283)
    at java.xml/com.sun.org.apache.xerces.internal.dom.ParentNode.nodeListGetLength(ParentNode.java:690)
    at java.xml/com.sun.org.apache.xerces.internal.dom.ParentNode.getLength(ParentNode.java:720)
    at com.github.lombrozo.xnav.SafeXml.children(SafeXml.java:41)
```

On the 5518th run, my test failed again. Any ideas? It took me a while to
understand where the problem is. Seriously, try to find it. It might be
exciting.

Well, I can't spend your time anymore. The problem is within this fancy
block of code:

```java
return Stream.iterate(0,idx->idx+1)
    .limit(length)
    .map(nodes::item)
    .map(n->new SafeXml(this.doc,n));
```

I personally enjoy using Java Stream API. It's short, concise, and powerful.
Moreover, it's **lazy**. It means that the actual iteration happens not in this
method, but when you call the `collect()` method. As you can see, in our case,
we don't call it. We just build the stream and return it.
So, the iteration happens somewhere outside, avoiding all the synchronization
we've added.

Hence, we need to collect all the children nodes eagerly. Here is how we can
do it with the `Collectors.toList()` method:

```java
return Stream.iterate(0,idx->idx+1)
    .limit(length)
    .map(nodes::item)
    .map(n->new SafeXml(this.doc,n))
    .collect(Collectors.toList())
    .stream();
```

Or by using the `Stream.Builder`:

```java
    public Stream<SafeXml> children() {
        synchronized (this.doc) {
            NodeList nodes = this.node.getChildNodes();
            int length = nodes.getLength();
            Stream.Builder<SafeXml> builder = Stream.builder();
            for (int i = 0; i < length; i++) {
                Node n = nodes.item(i);
                builder.accept(new SafeXml(this.doc, n));
            }
            return builder.build();
        }
    }
```

Or even use the plain `ArrayList`:

```java
    public Stream<SafeXml> children() {
        synchronized (this.doc) {
            NodeList nodes = this.node.getChildNodes();
            int length = nodes.getLength();
            List<SafeXml> children = new ArrayList<>(length);
            for (int i = 0; i < length; i++) {
                Node n = nodes.item(i);
                children.add(new SafeXml(this.doc, n));
            }
            return children.stream();
        }
    }
```


However, according to the benchmarks, the solution with `Stream.Builder` is the
fastest one:

```shell
Benchmark                        Mode  Cnt  Score   Error  Units
SafeXmlBenchmark.array           avgt       0.780          us/op
SafeXmlBenchmark.collect_stream  avgt       1.360          us/op
SafeXmlBenchmark.stream_builder  avgt       0.631          us/op
```

So, we've finally fixed the synchronization leakage and understood that plain 
using of Java Stream API might be rather tricky and require some additional
attention. It's still not clear how to prevent such issues statically. The only
way is to write tests and run them in parallel.