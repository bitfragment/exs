---
title: "Strip leading"
date: 2018-09-15
tags: py
---

Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}

## Description

Given an iterable and an item, strip any leading elements of the iterable
that are equivalent to that item. Return remaining data as an iterator.

The item may be a function, in which case strip leading elements of
the iterable that cause the function to return true when passed as
an argument.


## Notes

Use `itertools/dropwhile`.


## My solution

Here, a factory function provides the appropriate evaluation. 
Use `callable()` to detect a function passed as an argument.

```py
def test_item(iter, item):
    if callable(item):
        def test(first):
            return item(iter[first])
    else:
        def test(first):
            return item == iter[first]
    return test


def lstrip(iter, item):
    iter = list(iter)
    iterlen = len(iter)
    if not iterlen:
        return []

    test = test_item(iter, item)
    first = 0
    try:
        while test(first) and first < iterlen:
            first += 1
    except IndexError:
        pass

    return (item for item in iter[first:])
```


## Provided solution

Much more compact and elegant.

```py
from itertools import dropwhile

def lstrip(iterable, strip_value):
    if callable(strip_value):
        predicate = strip_value
    else:
        def predicate(value):
            return value == strip_value
    return dropwhile(predicate, iterable)
```
