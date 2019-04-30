---
title: "Count bits"
date: 2019-04-30
tags: java, core
---

Source: [Elements of Programming Interviews], Java Edition

[Elements of Programming Interviews]: https://elementsofprogramminginterviews.com/

Problem 4.0

## Solution

Until `x` == 0, mask `x` with `& 1` to determine if the least significant bit
is 1. If so, the result will be 1. This is the value to add to the number of
bits counted. Then, right-shift one bit, using the unsigned right-shift 
operator `>>>`, and continue.

Brute force solution. O(n), performing one computation per bit, with worst
case being 0xFFFFFFFF.

```java
public static int countBits(int x) {
    short bits = 0;
    while (x != 0)  {
        bits += (x & 1);
        x >>>= 1;
    }
    return bits;
}
```


## Tests

```java
assert countBits(0) == 0;
assert countBits(1) == 1;
assert countBits(2) == 1;
assert countBits(3) == 2;
assert countBits(255) == 8;
assert countBits(0xFFFFFFFF) == 32;
```
