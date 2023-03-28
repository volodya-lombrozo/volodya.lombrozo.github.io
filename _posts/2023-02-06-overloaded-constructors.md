---
layout: post
title: "Overloaded Constructors"
date: 2023-02-06
---

Not so long time ago I was working on the implementation of a new feature in
one of our projects and...

It's a well-known fact that professional Java developers tend to structure all
classes use the same schema across all `.java` files. It is similar to something
like:

```java
public class Circle {
    // Static class fields or also known static variables
    private static int pi = 3.14;

    // Instance fields or instance variables
    private int r;

    // Constructors
    Circle() {
        this.r = 1;
    }

    // Methods
    public int square() {
        return DeclarationOrder.pi * this.r * this.r;
    }
} 
```

That is all. When I'm and, I hope, all developers will read that class they
almost 100% won't scan entire class members to find desirable place. For
example, if I want to know "How I can create an object" - I will look the
code right above all variables declaration. If I want to know the object "
Protocol" I'll definitely scroll down after all variables and constructors.
Of course, you can mixe them up, and it will still work, but the other developer
or you on the next day definitely won't be happy because other developers
have some expectations towards your code.

Of course, it's not only the one type of stucturing the code. If you are using
linters like CheckStyle, SonarQube or PMD they will check not only main members
positions, but they will expect even stronger structure. For example, all
variables, constructors and methos have to keep the order of visibility.
In other words, all `public` fields have to be placed before `package-private`
fields and the last should be `private` members. For example, our class might
look like:

```java
public class Circle {
    // Static class fields or also known static variables
    private static int pi = 3.14;

    // Instance fields or instance variables
    private int r;

    // Constructors
    public Circle() {
        this(1);
    }

    private Circle(final int r) {
        this.r = r;
    }

    // Methods
    public int square() {
        return DeclarationOrder.pi * this.pow();
    }

    private int pow() {
        return this.r * this.r;
    }
} 
```

If you place methods in different order, the linter will definitely
tell you about it.

The other important example of class structure is Unit-Tests.
JUnit 5 test example for our `Circle` class would look like:

```java
public class CircleTest {

    private Circle circle;

    @BeforeEach
    public static setUp() {
        circle = new Circle();
    }

    @Test
    int calculatesSquare() {
        assertEquals(3.14, circle.square());
    }

    @AterEach
    public static tearDown() {
    }
} 
```

JUnit 5 executes the @BeforeEach methods before each test method, and @AfterEach
methods after each test method in the test class. So as long as
your @BeforeEach and @AfterEach methods are defined in the test class, they will
be executed in the appropriate order regardless of their position in the test
class. However, it's usually a good practice to keep the @BeforeEach
methods at the beginning of the test class and @AfterEach methods at the end of
the test class for better readability and maintainability of the test code.

Another noticeable example is Utility classes (despite that I believe that
Utility classes is pure evil). Utility classes often contain only static
methods and do not have any instance variables. They are used to
provide helper methods to other parts of the application such as formatting
strings or parsing dates. Developers often structure Utility classes in a way
that groups related methods together which makes it esier to discover related
methods.

For example Strings class from Google Guava library:
```java
public final class Strings {
  private Strings() {}

  public static String nullToEmpty(@CheckForNull String string) {
      // Method body
  }

  public static String emptyToNull(@CheckForNull String string) {
      // Method body
  }

  public static boolean isNullOrEmpty(@CheckForNull String string) {
      // Method body
  }

  public static String padStart(String string, int minLength, char padChar) {
      // Method body
  }
  
  public static String padEnd(String string, int minLength, char padChar) {
      // Method body
  }
  
  public static String repeat(String string, int count) {
     // Method body
  }

  public static String commonPrefix(CharSequence a, CharSequence b) {
      // Method body
  }

  public static String commonSuffix(CharSequence a, CharSequence b) {
      // Method body 
  }
 
  public static String lenientFormat(String template, Object... args) {
      // Method body
  }
  private static String lenientToString(Object o) {
      // Method body
  }
}

```

