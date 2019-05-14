---
title: "Number to string"
date: 2017-11-02
tags: java
---

Given a number `n`, return its string representation.

```
123 ⟹ '123'
123.456 ⟹ '123.456'
```

```java
public class numberToString {

    public static String numberToString(int n) {
        return String.valueOf(n);
    }

    public static String numberToString(double n) {
        return String.valueOf(n);
    }

}
```

## Tests

```java
import org.junit.*;
import static org.junit.Assert.*;

public class numberToStringTest {
    numberToString x = new numberToString();

    @Test
    public void testInt() {
        assertEquals(x.numberToString(123), "123");
    }

    @Test
    public void testDouble() {
        assertEquals(x.numberToString(123.456), "123.456");
    }

}
```
