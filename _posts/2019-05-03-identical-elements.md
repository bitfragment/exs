---
title: "Identical elements"
date: 2019-05-03
tags: python
---

Source: CheckiO: [All the Same]

[All the Same]: https://py.checkio.org/en/mission/all-the-same/


* TOC
{:toc}


## Description

Determine if all elements in the given list are equal.

Precondition: all elements of the input list are hashable.

```py
all_the_same([1, 1, 1]) == True
all_the_same([1, 2, 1]) == False
all_the_same(['a', 'a', 'a']) == True
all_the_same([]) == True
```


## My solution

```py
"""Identical elements."""

from typing import Any, Sequence, Set


def solution1(seq: Sequence[Any]) -> bool:
    """Return True if all elements are identical.

    This implementation uses `!=` for comparison.

    Args:
        seq: a sequence

    Returns:
        True if all elements are identical, or False otherwise.

    Examples:
        >>> solution1((1, 1, 1))
        True

    """
    if len(seq) < 2:
        return True
    first = seq[0]
    for elt in seq:
        if elt != first:
            return False
    return True


def solution2(seq: Sequence[Any]) -> bool:
    """Return True if all elements are identical.

    This implementation uses set membership for comparison.

    Args:
        seq: a sequence

    Returns:
        True if all elements are identical, or False otherwise.

    Examples:
        >>> solution2((1, 2, 3))
        False

    """
    if len(seq) < 2:
        return True
    seen: Set = {seq[0]}
    for elt in seq:
        if elt not in seen:
            return False
    return True


def interactive():
    """Line-oriented command interpreter."""
    from cmd import Cmd

    class MyPrompt(Cmd):
        def do_allsame(self, arg):
            """Return True if all values are identical.

            Examples:
                allsame 1 2 3 → False
                allsame 1 1 1 → True

            """
            print(solution1(tuple(arg.split())))

    print("'help' to see commands")
    MyPrompt().cmdloop()


if __name__ == "__main__":
    import sys

    if sys.argv[1] == "-i":
        interactive()
```


## Tests

```py
"""Tests for solution.py."""

from hypothesis import given
import hypothesis.strategies as st

from solution import solution1
from solution import solution2


def test_provided():
    """Tests provided with problem."""
    solutions = solution1, solution2
    for solution in solutions:
        assert solution([1, 1, 1]), "expected True for identical integers"
        assert not solution(
            [1, 2, 1]
        ), "expected False for non-identical integers"
        assert solution(
            ["a", "a", "a"]
        ), "expected True for identical characters"
        assert solution([]), "expected True for empty list"


def test_mytest():
    """My initial tests."""
    solutions = solution1, solution2
    for solution in solutions:
        assert solution(["foo"]), "expected True for single element"
        assert solution(
            ["foo", "foo", "foo"]
        ), "expected True for identical strings"
        assert not solution(
            ["foo", "bar", "baz"]
        ), "expected False for non-identical strings"


@given(st.lists(st.integers(), min_size=2, unique=True))
def test_property_test_nonidentical__elts(xs):
    """Hypothesis property test for non-identical integers."""
    solutions = solution1, solution2
    for solution in solutions:
        assert not solution(xs), "expected False for non-identical integers"


@given(st.lists(st.integers(min_value=1, max_value=1), min_size=2))
def test_property_test_identical_elts(xs):
    """Hypothesis property test for identical integers."""
    solutions = solution1, solution2
    for solution in solutions:
        assert solution(xs), "expected True for identical integers"


def test_worst_solution1(benchmark):
    """Benchmark solution1() with sample worst-case data."""
    data = ["x"] * 10000000 + ["y"]
    result = benchmark(lambda: solution1(data))
    assert not result, "expected False for non-identical characters"


def test_worst_solution2(benchmark):
    """Benchmark solution2() with sample worst-case data."""
    data = ["x"] * 10000000 + ["y"]
    result = benchmark(lambda: solution2(data))
    assert not result, "expected False for non-identical characters"
```


## Benchmarking

```
---------------------- benchmark: 2 tests ---------------------
Name (time in ms)            Mean              Median          
---------------------------------------------------------------
test_worst_solution2     375.6234 (1.0)      372.8045 (1.0)    
test_worst_solution1     391.7945 (1.04)     392.0774 (1.05)   
---------------------------------------------------------------
```

```
---------------------- benchmark: 2 tests ---------------------
Name (time in ms)            Mean              Median
---------------------------------------------------------------
test_worst_solution2     375.8267 (1.0)      378.9059 (1.0)
test_worst_solution1     390.1238 (1.04)     388.0440 (1.02)
---------------------------------------------------------------
```

```
---------------------- benchmark: 2 tests ---------------------
Name (time in ms)            Mean              Median
---------------------------------------------------------------
test_worst_solution2     380.6648 (1.0)      378.6950 (1.0)
test_worst_solution1     395.4856 (1.04)     394.6585 (1.04)
---------------------------------------------------------------
```
