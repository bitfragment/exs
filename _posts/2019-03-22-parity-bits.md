---
title: "Parity bits"
date: 2019-03-22
tags: python
---

Source: Ben Stephenson, [The Python Workbook](https://www.springer.com/us/book/9783319142395), [Exercise 68: Parity Bits](https://link.springer.com/chapter/10.1007/978-3-319-14240-1_3)


* TOC
{:toc}


## Description

"Write a program that computes the parity bit for groups of 8 bits entered by
the user using even parity."

```py
>>> solution('10010011')
'0'
>>> solution('10010010')
'1'
```


## My solution

```py
"""Parity bits."""

from cmd import Cmd


def solution(bp: str) -> str:
    """Return '1' or '0' to compute even parity of `bp`.

    `bp` represents an 8-bit pattern using '1' and '0'.

    >>> solution('10010011')
    '0'
    >>> solution('10010010')
    '1'
    """
    return "1" if bp.count("1") % 2 else "0"


class MyPrompt(Cmd):
    """Line-oriented command interpreter."""

    def do_solution(self, arg):
        """Handle input for `solution()`."""
        print("Input error" if len(arg) != 8 else solution(arg))


if __name__ == "__main__":
    MyPrompt().cmdloop()
```


## Tests

```py
"""Tests for solution."""

from hypothesis import given
from hypothesis.strategies import characters, text
from solution import solution


def test_solution_basic():
    """Hard-coded test data."""
    assert solution("10010011") == "0", "failed on even"
    assert solution("10010010") == "1", "failed on odd"


def test_solution_range():
    """Generated data."""

    def bp(i):
        s = "1" * i
        s += "0" * (8 - i)
        return s

    even = list(range(0, 9, 2))
    odd = list(range(1, 8, 2))

    for x in (bp(i) for i in even):
        assert solution(x) == "0", "failed on even"

    for x in (bp(i) for i in odd):
        assert solution(x) == "1", "failed on odd"


@given(
    s=text(
        alphabet=characters(min_codepoint=48, max_codepoint=49),
        min_size=8,
        max_size=8,
    )
)
def test_solution_prop(s):
    """Property test."""

    def count(str, chr):
        """Implement to avoid reproducing the solution."""
        return len([c for c in str if c == chr])

    assert (
        solution(s) == "1" if count(s, "1") % 2 else solution(s) == "0"
    ), "failed property test"
```
