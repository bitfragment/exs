---
title: "Unique items"
date: 2018-09-01
tags: python
---

Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}

## Description

Return all unique items in an iterable. Return a lazy iterator (generator);
work with non-hashable types; optimize for both hashable and non-hashable types.

```py
>>> list(uniques_only([1, 2, 2, 1, 1, 3, 2, 1]))
[1, 2, 3]
>>> nums = [1, -3, 2, 3, -1]
>>> squares = (n**2 for n in nums)
>>> list(uniques_only(squares))
[1, 9, 4]
>>> list(uniques_only([['a', 'b'], ['a', 'c'], ['a', 'b']]))
[['a', 'b'], ['a', 'c']]
```

## Notes

Solution involves using `set()` to optimize lookup of hashable items, and a list
to accumulate them if you need to preserve order (because a set is unordered).

## My solution

My first thought was to convert the input list to a `set()` to eliminate
duplicates. However, sets are unordered and the provided tests required
ordered output. So I used a list to accumulate  values, checking the list of
accumulated values to avoid adding the same value twice. Checking the entire
list of accumulated values would make this slow for sizable input iterables,
but that's as far as I got on a busy day.

```py
def uniques_only(iter):
    result = []
    for item in iter:
        if item not in result:
            result.append(item)
            yield item
```

## Provided solutions

One possibility is to add accumulated values to a set. Sets use hashes for
lookup, so this will likely be faster than a list for large inputs.
Separately, a list is used to accumulate the output, because sets are
unordered.

However, this solution will not work with non-hashable types.

```py
def uniques_only(iterable):
    seen = set()
    items = []
    for item in iterable:
        if item not in seen:
            items.append(item)
            seen.add(item)
    return items
```

An optimized solution for both hashable and non-hashable types would require
tracking both separately (hashables in a set, non-hashables in a list), and
either (1) checking each item to see if it is an instance of `Hashable` (use
`isinstance` for this)  or (2) more Pythonically, letting  an exception
determine what happens: try to add it to a set, and if that throws a
`TypeError` then add it to a list.
