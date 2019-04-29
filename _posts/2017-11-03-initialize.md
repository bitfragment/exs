---
title: "Initialize"
date: 2017-11-03
tags: java
---

## Source

Source: variation on [CodeChef] problem [NITIKA]: "Whats in 
the Name"

[CodeChef]: http://codechef.com/
[NITIKA]: https://www.codechef.com/problems/NITIKA


## Description

Given a string `fullname` representing a person's full name:

- if `fullname` contains only one name, capitalize it
- if `fullname` contains more than one name, replace all but
  the final name with an initial followed by a period.

```
eliot ⟹ Eliot
thomas eliot ⟹ T. Eliot
thomas stearns eliot ⟹ T. S. Eliot
```


## My solution

```java
class initialize {
    
    public static String initialize(String fullname) {
        String[] names = fullname.split(" ");
        int nameslen = names.length;
        String last = names[nameslen - 1];
        names[nameslen - 1] = last.substring(0, 1).toUpperCase() +
                              last.substring(1);
        for (int i = 0; i < nameslen - 1; i++) {
            String curr = names[i];
            names[i] = curr.substring(0, 1).toUpperCase() + ".";
        }
        return String.join(" ", names);
    }
    
}
```


## Tests

```java
import org.junit.*;
import static org.junit.Assert.*;

public class initializeTest {
    initialize x = new initialize();

    @Test
    public void testSingleName() {
        assertEquals(x.initialize("eliot"), "Eliot");
    }
    
    @Test
    public void testOneInitial() {
        assertEquals(x.initialize("thomas eliot"), "T. Eliot");
    }
    
    @Test
    public void testTwoinitials() {
        assertEquals(x.initialize("thomas stearns eliot"), "T. S. Eliot");
    }
}
```
