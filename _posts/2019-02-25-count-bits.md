---
title: "Count bits"
date: 2019-02-25
tags: c++
---

Source: [Elements of Programming Interviews], C++

[Elements of Programming Interviews]: https://elementsofprogramminginterviews.com/

Problem 4.0

## Solution

Until `x` == 0, mask `x` with `& 1` to determine if the least significant bit
is 1. If so, the result will be 1. This is the value to add to the number of
bits counted. Then, right-shift one bit and continue.

Brute force solution. O(n), performing one computation per bit, with worst
case being 0xFFFFFFFF.

```cpp
short count_bits(unsigned int x) {
    short bits = 0;
    while (x) {
        bits += x & 1;
        x >>= 1;
    }

    return bits;
}
```


## Tests

```cpp
#include "catch.hpp"
#include "solution.cpp"


TEST_CASE("Count bits in an unsigned integer", "[count_bits]") {
    CHECK(count_bits(0) == 0);
    CHECK(count_bits(1) == 1);
    CHECK(count_bits(2) == 1);
    CHECK(count_bits(3) == 2);
    CHECK(count_bits(255) == 8);
    CHECK(count_bits(0xFFFFFFFF) == 32);
}
```
