---
title: "Cache array elements in hash table"
date: 2019-02-22
tags: python
---

Source: LeetCode, [Two Sum]

[Two Sum]: https://leetcode.com/problems/two-sum/

"Given an array of integers, return indices of the two numbers such that
they add up to a specific target. You may assume that each input would
have exactly one solution, and you may not use the same element twice."

```py
>>> solution((1, 3, 5, 7, 9), 14)
(2, 4)
```

Use a dictionary to cache the index value of each complement
(the difference between the target and the current value), so
we can return that value immediately. This be done in a single
pass.


```py
from typing import Dict, Tuple


def solution(ns: Tuple[int, ...], target: int) -> Tuple[int, int]:
    """Return indexes of the first two elements whose sum is `target`.

    Each input must have only one solution, and the same element may
    not be used twice. If there is no solution, return (0, 0).

    >>> solution((1, 3, 5, 7, 9), 14)
    (2, 4)
    """
    compls: Dict[int, int] = {}
    for i in range(len(ns)):
        c: int = target - ns[i]
        if c in compls:
            return compls[c], i
        compls[ns[i]] = i
    return (0, 0)
```


## Tests

```py
from hypothesis import given
import hypothesis.strategies as st

from solution import solution


def test_solution_mytest_nonsoluble():
    """Nonsoluble."""
    assert solution((1, 2), 4) == (0, 0), "failed on nonsoluble"


def test_solution_mytest_soluble_begin():
    """Solution is in first two elements."""
    assert solution((1, 2, 3), 3) == (0, 1)


def test_solution_mytest_soluble_end():
    """Solution is in last two elements."""
    assert solution((1, 2, 3, 4, 5), 9) == (3, 4)


def test_solution_mytest_soluble_other():
    """Solution is in other elements."""
    assert solution((1, 3, 5, 7, 9), 14) == (2, 4)


def test_solution_provided():
    """Test provided with problem."""
    assert solution((2, 7, 11, 15), 9) == (0, 1)


@given(
    ns=st.tuples(
        st.integers(0, 3),
        st.integers(4, 7),
        st.integers(8, 11),
        st.integers(12, 15),
        st.integers(16, 19),
    )
)
def test_prop_solution(ns):
    """Property test.

    Given tuples of unique integers, examine the last two."""
    target = sum(ns[-2:])
    assert solution(ns, target) == (3, 4)
```
