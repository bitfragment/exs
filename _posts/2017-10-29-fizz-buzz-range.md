---
title: "Fizz Buzz with range"
date: 2017-10-29
tags: python
---

## Source

Source: [CodeEval]: [Fizz Buzz]

[CodeEval]: https://www.codeeval.com/
[Fizz Buzz]: https://www.codeeval.com/open_challenges/1/

Given three positive integers *x*, *y*, *n*, in which
*x* is the first divisor, *y* is the second divisor, and *n* is a
maximum value, return an array of values 1 through *n* in which:

- numbers divisible by *x* are replaced with "F"
- numbers divisible by *y* are replaced with "B"
- numbers divisible by both *x* and *y* are replaced with "FB"


```
3, 5, 10 ⟹ [1, 2, "F", 4, "B", "F", 7, 8, "F", "B"]

2, 7, 15 ⟹ [1, "F", 3, "F", 5, "F", "B", "F", 9, "F", 11, "F", 13, "FB", 15]
```


## My solutions: without range

Imperative:

```py
def fizzbuzz(x, y, n):
    result = ''
    if not n % x:
        result += 'F'
    if not n % y:
        result += 'B'
    if not result:
        result = n
    return result
```


Functional solution using lambdas and ternary conditional 
expressions:

```py
def fizzbuzz(x, y, n):
    fizz = lambda x, y, n: 'F' if not n % x else ''
    buzz = lambda x, y, n: 'B' if not n % y else ''
    result = fizz(x, y, n) + buzz(x, y, n)
    return result if result else n
```


## My solutions: including range

Functional solution using list comprehension:

```py
def fizzbuzz_range(x, y, n):
    return [fizzbuzz(x, y, z) for z in range(1, n + 1)]
```


## Tests

```py
import pytest
from fizzbuzz_range import *

def test_fizzbuzz_fizz():
    """Should return 'F' when n is divisible by x only"""
    assert fizzbuzz(3, 5, 6) == 'F'

def test_fizzbuzz_buzz():
    """Should return 'B' when n is divisible by y only"""
    assert fizzbuzz(3, 5, 10) == 'B'

def test_fizzbuzz_fizzbuzz():
    """Should return 'FB' when n is divisible by both x and y"""
    assert fizzbuzz(3, 5, 15) == 'FB'

def test_fizzbuzz_none():
    """Should return n when not divisible by either x or y"""
    assert fizzbuzz(3, 5, 2) == 2

def test_fizzbuzz_range():
    """Should return the correct result"""
    assert fizzbuzz_range(3, 5, 10) == \
        [1, 2, "F", 4, "B", "F", 7, 8, "F", "B"]
    assert fizzbuzz_range(2, 7, 15) == \
        [1, "F", 3, "F", 5, "F", "B", "F", 9, "F", 11, "F", 13, "FB", 15]
```
