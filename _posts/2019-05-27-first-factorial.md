---
title: "First factorial"
date: 2019-05-27
tags: python
src-title: "Coderbyte: First factorial"
src-url: https://www.coderbyte.com/editor/guest:First%20Factorial:Python
---

## Description

> Have the function `FirstFactorial(num)` take the `num` parameter being
> passed and return the factorial of it. For example: if num = 4, then your
> program should return (4 * 3 * 2 * 1) = 24. For the test cases, the range will
> be between 1 and 18 and the input will always be an integer. 

## My solutions

```py
def solution1(num): 
    first = 1
    for factor in range(1, num + 1):
        first *= factor
    return first
```

```py
from functools import reduce

def solution2(num):
    return reduce(lambda x, y: x * y, range(1, num + 1))
```
