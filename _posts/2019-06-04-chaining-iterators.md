---
title: Chaining iterators
date: 2019-06-04
tags: python
src-author: "Dan Bader"
src-title: "Python Tricks: The Book"
src-url: https://realpython.com/products/python-tricks-book/
---

§6.7 Iterator Chains

## Using generator functions

```py
from typing import Iterator

def f1():
    for x in range(10):
        yield x

def f2(xs: Iterator[int]):
    for x in xs:
        yield 2 * x

assert list(f2(f1())) == [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

## Using generator expressions

```py
a = (i for i in range(10))
b = (2 * x for x in a)
assert list(b) == [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```
