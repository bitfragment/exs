---
title: "Multiple maximums"
date: 2018-08-22
tags: py
---

Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}

## Description

Return all maximum values in an iterable.

```
>>> multimax([1, 4, 2, 4, 3])
[4, 4]
```


## Notes

Solution involved using using a lambda function as a keyword argument.


## My solution

```py
def multimax(iter, key=lambda x: x):
    iter, max = list(iter), []
    if not len(iter):
        return max
    max.append(iter[0])
    for item in iter[1:]:
        kitem, kmax = key(item), key(max[0])
        if kitem > kmax:
            max = [item]
        elif kitem == kmax:
            max.append(item)
    return max
```


## Provided solution

This solution does not invoke the `key` function on the current
maximum value during each iteration, as mine does unnecessarily.

```py
def multimax(iterable, key=lambda x: x):
    max_key = None
    maximums = []
    for item in iterable:
        k = key(item)
        if k == max_key:
            maximums.append(item)
        elif not maximums or k > max_key:
            maximums = [item]
            max_key = k
    return maximums
```
