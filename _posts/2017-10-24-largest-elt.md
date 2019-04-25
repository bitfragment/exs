---
title: "Largest element"
date: 2017-10-24
tags: python
---

## Source

Adrian Neumann, [Simple Programming Problems](https://adriann.github.io/programming_problems.html), Lists, Strings problem 1: "Write a function that returns the largest element in a list."


## Description

Define a generic function that, given an array, returns the
largest element it contains.

Treat numeric values literally: `99` is the largest element in the
array `[0, 1, 99]`. Measure the size of an associative array (hash map)
by the number of keys it contains. If two or more elements are the 
largest, return the first largest element. If passed data that is not 
an array, or if the array is empty, return null.

```
0 ⟹ null

[] ⟹ null

[null, undefined, 1] ⟹ 1

[0, 999, 23] ⟹ 999

['foo', 'bar'] ⟹ 'foo'

[null, undefined, 'foobar', 5, [1, 2, 3], {a: 1, b: 2}] ⟹ 'foobar'
```


## My solution

```py
def getsize(elt):
    if not elt:
        return 0
    for t in [int, float, complex]:
        if isinstance(elt, t):
            return elt
    for t in [str, list]:
        if isinstance(elt, t):
           return len(elt)
    if isinstance(elt, dict):
        return len(list(elt.keys()))


def largest(lst):
    if not lst or not len(lst):
        return None
    relt = None
    rsize = 0
    for elt in lst:
        s = getsize(elt)
        if s > rsize:
            rsize = s
            relt = elt
    return relt
```


## Tests

```py
import pytest
from largest import *


# getsize()

def test_getsize_none():
    """Should return 0 if passed None."""
    assert getsize(None) == 0

def test_getsize_num():
    """Should return the same value if given a numeric value."""
    assert getsize(1) == 1
    assert getsize(1.1) == 1.1
    assert getsize(1j) == 1j

def test_getsize_str():
    """Should return the length of a string."""
    assert getsize('foo') == 3

def test_getsize_list():
    """Should return the length of a list."""
    assert getsize([1, 2, 3]) == 3

def test_getsize_dict():
    """Should return the length of a dictionary's keys."""
    assert getsize({'a': 1, 'b': 2, 'c': 3}) == 3


# largest(): boundaries

def test_largest_none():
    """Should return None if passed anything but a list."""
    assert largest(None) == None
    assert largest(0) == None

def test_largest_empty():
    """Should return None if passed an empty list."""
    assert largest([]) == None

def test_largest_nolength():
    """Should treat elements without a length as having length 0."""
    assert largest([None, [], 1]) == 1


# largest(): expected values

def test_largest_num():
    """Should return the largest number in a list of numbers."""
    assert largest([0, 999, 23]) == 999

def test_longest_string():
    """Should return the longest string in a list of strings."""
    assert largest(['foo', 'barbaz', 'qux']) == 'barbaz'

def test_longest_list():
    """Should return the longest list in a list of lists."""
    assert largest([[0], [1, 2, 3], [4, 5]]) == [1, 2, 3]

def test_largest_dict():
    """Should return the largest dictionary in a list of dictionaries."""
    a, b, c = {'a': 0}, {'a': 0, 'b':1, 'c': 2}, {'a': 0, 'b': 1}
    assert largest([a, b, c]) == b


# largest(): expected behavior

def test_first_of_two_largest():
    """Should return the first of two largest elements."""
    a, b, c = [0], [1, 2], [3, 4]
    assert largest([a, b, c]) == b

def test_mixed_data():
    """Should correctly handle lists of mixed data types."""
    a = None
    b = 5
    c = 'foo'
    d = [1, 2, 3, 4]
    e = {'a': 1, 'b': 2}
    assert largest([a, b, c, d, e]) == b

    d = [1, 2, 3, 4, 5, 6]
    assert largest([a, b, c, d, e]) == d

    c = 'foobarbaz'
    assert largest([a, b, c, d, e]) == c
```
