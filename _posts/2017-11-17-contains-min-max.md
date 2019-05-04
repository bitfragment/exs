---
title: "Contains min-max"
date: 2017-11-17
tags: python
---

## Source

HackerEarth: Basic Programming: Implementation: Basics of Implementation: [Min-Max]

[Min-Max]: https://www.hackerearth.com/practice/basic-programming/implementation/basics-of-implementation/practice-problems/algorithm/min-max-3/


## Description

Return true if array contains integer sequence

Given an array of integers, return true if the array contains the
entire integer sequence from the minimum to the maximum value in
the array.

```
4 2 1 3 5 6 ⟹ true
0 2 1 3 5 6 ⟹ false
```


## My solution

```py
def contains_minmax(lst):
    for i in range(min(lst) + 1, max(lst)):
        if i not in lst:
            return False
    return True
```

## Tests

```py
from contains_minmax import contains_minmax

def test_contains_minmax_true():
    """Should return true for array containing integer sequence."""
    assert contains_minmax([4, 2, 1, 3, 5, 6])

def test_contains_minmax_false():
    """Should return false for array not containing integer sequence."""
    assert not contains_minmax([0, 2, 1, 3, 5, 6])
```
