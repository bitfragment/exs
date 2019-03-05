---
title: "Compute centenary"
date: 2019-03-04
tags: python
---

Source: Practice Python, [Character Input]

[Character Input]: https://www.practicepython.org/exercise/2014/01/29/01-character-input.html


## Solution

```py
from datetime import datetime


def solution(age1: int, age2: int) -> int:
    """Return the year when a person aged `age1` will turn `age2`.

    >>> solution(35, 100) # assumes year is 2019
    2084
    """
    return age2 - age1 + datetime.today().year
```


## Tests

```py
from hypothesis import given
import hypothesis.strategies as st

from solution import solution


def _test_solution1(year, age1, age2):
    "Helper function to test inputs to `solution()`."
    assert (
        solution(age1, age2) == age2 + year - age1
    ), f"failed on inputs {age1}, {age2}"


def test_solution1_mytest():
    "My tests for `solution()`."
    year = 2019
    age2 = 100
    for age1 in range(100):
        _test_solution1(year, age1, age2)


@given(age1=st.integers(), age2=st.integers())
def test_solution1_mytest_prop(age1, age2):
    "My Hypothesis property tests for `solution()`."
    year = 2019
    age2 = 100
    _test_solution1(year, age1, age2)
```
