---
title: "Endianness"
date: 2018-08-27
tags: c, core
published: true
---

Source: [CS:APP3e, Bryant and O'Hallaron](http://csapp.cs.cmu.edu/), 
practice problem 2.5

* TOC
{:toc}

## Description

Verify endianness by checking the byte ordering of a four-byte integer value.

## Notes

On a little-endian system (e.g., Intel x86), the low-order (least
significant) byte will be stored first, so `0x12345678` will be  represented
by the byte order `78 56 34 12`.

On a big-endian system (e.g., Motorola 68000), the low-order (least
significant) byte will be stored last, so `0x12345678` will be  represented by
the byte order `12 34 56 78`.

## Solution

Create a pointer to the first byte in a four-byte integer constant by casting
to `unsigned char`. Dereference and increment this pointer, checking its value
each time and returning false if byte order is not little-endian.

```c
#include <assert.h>

int is_little_endian() {
    int x = 0x12345678;
    unsigned char * byte_ptr = (unsigned char *) &x;
    if (*byte_ptr++ != 0x78) return 0;
    if (*byte_ptr++ != 0x56) return 0;
    if (*byte_ptr++ != 0x34) return 0;
    if (*byte_ptr != 0X12) return 0;
    return 1;
}

int main(void) {
    assert(is_little_endian());
    return 0;
}
```
