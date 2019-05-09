---
title: "Balanced parentheses"
date: 2017-10-21
tags: python
---

## Source

Advent of Code [2015](http://adventofcode.com/2015),
[Day 1: Not Quite Lisp](http://adventofcode.com/2015/day/1)


## Description

Given a string `s`, scan it from left to right. Setting `n` to an
initial value of `0`, for each occurrence of an opening parenthesis
`(`, increment `n` by one. For each occurrence of a closing 
parenthesis `)`, decrement `n` by one. Return `n`. 
(A return value of 0 indicates that parentheses are "balanced.")

```
(())    ⟹ 0
()()    ⟹ 0
(((     ⟹ 3
(()(()( ⟹ 3
))((((( ⟹ 3
())     ⟹ -1
))(     ⟹ -1
)))     ⟹ -3
)())()) ⟹ -3
```


## My solution

```py
def parens(s):
    ps = {"(": 1, ")": -1}
    return sum(ps[c] for c in list(s))
```


## Tests

```py
import pytest
from parens import *

a = ['(())', '()()'] # 0
b = ['(((', '(()(()(', '))((((('] # 3
c = ['())', '))('] # -1
d = [')))', ')())())'] # -3

def test_parens_balanced():
    """Should return 0."""
    for elt in a:
        assert parens(elt) == 0

def test_parens_pos():
    """Should return positive values."""
    for elt in b:
        assert parens(elt) == 3

def test_parens_neg():
    """Should return negative values."""
    for elt in c:
        assert parens(elt) == -1
    for elt in d:
        assert parens(elt) == -3
```
