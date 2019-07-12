---
title: QWERTY array
date: 2017-03-11
tags: python
---

Type by coordinates of characters in a two-dimensional
QWERTY array. Also mimic toggle action of shift key.

```py
from re import search


def getc(key, kbd):
    [x, y] = [key[0], key[1]]
    return kbd[y][x]


def nextc(idx, n, keys, kbd):
    return None if len(keys) <= idx + n else getc(keys[idx + n], kbd)


def isi(idx, keys, kbd):
    curr = getc(keys[idx], kbd)
    next = lambda n: nextc(idx, n, keys, kbd)
    span = ''.join([x for x in [curr, next(1), next(2)] if x])
    return search('\Wi(\W|$)', span) != None


def typeout(keys, kbd):
    [result, capson] = ['', True]
    for idx, key in enumerate(keys):
        char = getc(key, kbd)
        result += char.upper() if capson else char
        if nextc(idx, 1, keys, kbd) == 'i' and isi(idx, keys, kbd):
            capson = True
        if char in ['.', '!', '?']:
            capson = True
        if char.isalpha():
            capson = False
    return result
```


## Tests

```py
kbd = [
    ['q', 'w', 'e', 'r', 't', 'y', 'u', 'i', 'o', 'p'],
    ['a', 's', 'd', 'f', 'g', 'h', 'j', 'k', 'l', ';', "'"],
    ['z', 'x', 'c', 'v', 'b', 'n', 'm', ',', '.', '?'],
    [ '',  '', ' ', ' ', ' ', ' ', ' ',  '',  '', '']
]


def test_typeout():
    keys = [(5, 1), (2, 0), (8, 1), (8, 1), (8, 0), (5, 3), (1, 0), (8, 0), (3, 0), (8, 1), (2, 1)]
    assert typeout(keys, kbd) == 'Hello world'

    keys = [(5, 1), (2, 0), (8, 1), (8, 1), (8, 0), (8, 2), (5, 3), (1, 0), (8, 0), (3, 0), (8, 1), (2, 1)]
    assert typeout(keys, kbd) == 'Hello. World'

    keys = [(5, 1), (2, 0), (8, 1), (8, 1), (8, 0), (5, 3), (1, 0), (8, 0), (3, 0), (8, 1), (2, 1), (5, 3), (7, 0), (5, 3), (8, 1), (8, 0), (3, 2), (2, 0), (5, 3), (5, 0), (8, 0), (6, 0)]
    assert typeout(keys, kbd) == 'Hello world I love you'
```
