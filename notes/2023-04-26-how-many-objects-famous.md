---
layout: post
title: "How many objects simple application creates?"
date: 2023-04-26
---

# How many objects?

# Tools for testing and profiling

YourKit and apache-jmeter.

Filtered yourkit related objects, methods, classes.

# Apache Tomcat

The first application that we age going to test is the plain old Apache Tomcat.
It's a very popular web server and servlet container that is very often used in
production. Since it's written in Java it makes it a perfect testing aim.

We are going to get one of the latest versions of Tomcat, for example,
the [Tomcat 10](https://tomcat.apache.org/download-10.cgi).

Since Tomcat has pre-installed examples, we are going to use them for our
test. Specifically, we are going to call simple http servlet that returns
a simple html page:

```html
http://localhost:8080/examples/servlets/servlet/RequestInfoExample/test
```

Then we will give a jmeter a task to call this servlet each 50 ms during 1
minute.

The received results are the next:

// no results yet - I have to understand how to capture results exactly for 1
// minute

# Spring MVC

Another good example of application that we can run is Spring MVC Rest service.
Since it should create many objects for each request body.

https://github.com/spring-guides/gs-rest-service

# Apache Kafka

# Apache Derby

* Total objects created: 935622

* Total constructors invoked (<init>):  1614040
* Total methods invoked: 61683273
* Time spent in constructors: ~ 9981,6 ms
* Time spent in all methods: ~ 292903,9 ms
* Final percent: 3%

# H2

The next noticeable application that we can test is H2 database. It's a very
popular embedded database that is used in many applications. It's also written
in Java. You can read about it more on
the [official site](https://www.h2database.com).

We are going to use the latest version of H2 database `2.1.214` which could
be downloaded from
the [site](https://www.h2database.com/html/download-archive.html).

As in all other examples we will use jmeter to make some load on the database.
Instead of plain HTTP in that case we will use JDBC connection to the database
and execute some
simple [JDBC query](https://jmeter.apache.org/usermanual/component_reference.html#JDBC_Request).

First of all we will init the DB with a test table:

```sql
CREATE TABLE IF NOT EXISTS TEST(ID INT PRIMARY KEY auto_increment, NAME VARCHAR(255));
```

Then we will insert some data into the table:

```sql
INSERT INTO TEST (NAME) VALUES('${__RandomString(10,abcdefghijklmnopqrstuvwxyz)}');
```

`${__RandomString(10,abcdefghijklmnopqrstuvwxyz)}` - is JMeter placeholder which
will insert random sting to the `NAME` value.

You can find the full jmeter test plan [here](/todo).

The received results are the next:

* Objects allocations - empty - reason is unknown.
* Total constructors invoked (\<init\>):  1191
* Total methods invoked: 26141
* Time spent in constructors: ~ 11,9 ms
* Time spent in all methods: ~ 215,5 ms
* Final percent: 5%

# Apache Cassandra

The next application that we are going to test is Apache Cassandra. It's a very
popular distributed database that is used in many applications.

We can't start profiling of that application, because it requires java 11 and
what is more important, they use `-XX:+PerfDisableSharedMem` option set to
prevent long pauses which makes JVM invisible to `jps`, `jstat` and other
profiling tools like YourKit.

# Takes framework
