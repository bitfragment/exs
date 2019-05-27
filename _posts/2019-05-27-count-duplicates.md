---
title: "Count duplicates"
date: 2019-05-27
tags: python
src-title: "Codewars: Counting Duplicates"
src-url: https://www.codewars.com/kata/counting-duplicates
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

> Write a function that will return the count of distinct case-insensitive
> alphabetic characters and numeric digits that occur more than once in the
> input string. The input string can be assumed to contain only alphabets
> (both uppercase and lowercase) and numeric digits.


## My solution

```py

from collections import Counter
from typing import Dict


def solution1(s: str = "") -> int:
    """Solution using a dictionary to store counts.

    Args:
        s: a string

    Returns:
        An integer representing a count of duplicates.

    Examples:
        >>> solution1("abba")
        2

    """
    counts: Dict[str, int] = {}
    for c in s.lower():
        counts[c] = counts[c] + 1 if c in counts else 1
    return sum(1 for x in counts.values() if x > 1)


def solution2(s: str = "") -> int:
    """Solution using `collections.Counter`.

    Args:
        s: a string

    Returns:
        An integer representing a count of duplicates.

    Examples:
        >>> solution1("abba")
        2

    """
    counts = Counter(s.lower()).values()
    return sum(1 for x in counts if x > 1)
```

## Tests

```py
import pytest
from random import randint
from solution import solution1, solution2


@pytest.mark.xfail
def test_input():
    """Should fail if passed argument of inappropriate type.

    Solutions perform no input validation.
    """
    for solution in solution1, solution2:
        solution(1)
        solution(1.23)
        solution([])
        solution({})


def test_return_type():
    """Should return value of type integer."""
    for solution in solution1, solution2:
        assert isinstance(
            solution(), int
        ), "expected return type to be integer"


def test_default_arg():
    """Should return 0 if invoked with no argument."""
    for solution in solution1, solution2:
        assert not solution(), "expected 0 for function call with no argument"


def test_empty_string():
    """Should return 0 for argument of empty string."""
    for solution in solution1, solution2:
        assert not solution(""), "expected 0 for empty string"


def test_one_char():
    """Should return 0 for single character."""
    for solution in solution1, solution2:
        assert solution("a") == 0, "expected 0 for single character"


def test_two_different():
    """Should return 0 for two different characters."""
    for solution in solution1, solution2:
        assert solution("ab") == 0, "expected 0 for two different characters"


def test_two_same():
    """Should return 1 for two identical characters."""
    for solution in solution1, solution2:
        assert solution("aa") == 1, "expected 1 for two identical characters"


def test_two_same_case():
    """Should treat characters case-insensitively."""
    for solution in solution1, solution2:
        assert (
            solution("aA") == 1
        ), "expected 1 (argument should be treated case-insensitively)"


def test_duplication_begins():
    """Should recognize duplication at beginning of string."""
    for solution in solution1, solution2:
        assert (
            solution("aab") == 1
        ), "expected 1 (duplication at beginning of string)"


def test_duplication_ends():
    """Should recognize duplication at end of string."""
    for solution in solution1, solution2:
        assert (
            solution("abb") == 1
        ), "expected 1 (duplication at end of string)"


def test_duplication_middle():
    """Should recognize duplication in middle of string."""
    for solution in solution1, solution2:
        assert (
            solution("abbc") == 1
        ), "expected 1 (duplication in middle of string)"


def test_range():
    """Should recognize randomly generated ranges of duplications."""
    mindup, maxdup = 2, 1000
    n1, n2, n3 = (randint(mindup, maxdup) for _ in range(3))
    s = f"{'b'*n1}{'c'*n2}{'d'*n3}"
    s = "a" + s if randint(0, 1) else s  # single-char prefix
    s += "e" if randint(0, 1) else ""  # single-char suffix
    for solution in solution1, solution2:
        assert (
            solution(s) == 3
        ), "expected 3 (randomly generated range of duplications)"


@pytest.mark.parametrize(
    "test_input, expected",
    [
        ("abcde", 0),
        ("aabbcde", 2),
        ("aabBcde", 2),
        ("indivisibility", 1),
        ("Indivisibilities", 2),
        ("aA11", 2),
        ("ABBA", 2),
    ],
)
def test_provided_tests(test_input, expected):
    """Should pass tests provided with exercise."""
    for solution in solution1, solution2:
        assert (
            solution(test_input) == expected
        ), f"expected {expected} (test provided with exercise)"


def _benchmark_setup():
    """Create benchmarking data."""
    s = "a" * 1_000_000 + "b"
    return (s,), {}


def _make_benchmark(solution):
    """Create a benchmarking function."""

    def f(benchmark):
        """Benchmark the solution."""
        result = benchmark.pedantic(solution, setup=_benchmark_setup)
        assert result == 1, "expected 1 (benchmarking test)"

    return f


test_benchmark_solution1 = _make_benchmark(solution1)

test_benchmark_solution2 = _make_benchmark(solution2)
```

## Benchmark results

```sh
2019-05-27
------------------------ benchmark: 2 tests -----------------------
Name (time in ms)                Mean              Median          
-------------------------------------------------------------------
test_benchmark_solution2      89.0296 (1.0)       89.0296 (1.0)    
test_benchmark_solution1     190.0885 (2.14)     190.0885 (2.14)   
-------------------------------------------------------------------
```
