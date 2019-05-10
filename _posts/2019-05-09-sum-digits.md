---
title: Sum digits
date: 2019-05-09
tags: python
src-title: "Code Abbey: Sum of digits"
src-url: https://www.codeabbey.com/index/task_view/sum-of-digits
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

> In this task you will be given several numbers and asked to calculate their sums of digits.
>
> Important: while many programming languages have built-in functions to convert numbers to strings (from which digits could be extracted), you should not use this (since your goal is to learn some programming tricks).
>
> Instead you need to implement algorithm with repetitive division by 10 (base of numeral system) and summing up the remainders.

## My solution

```py
import sys


def sum_digits(x: int) -> int:
    """Sum the digits in `x`.

    Args:
        x: an integer >= 0

    Returns:
        An integer representing the sum of the digits in `x`.

    Examples:
        >>> sum_digits(123)
        6

    """
    result = 0
    quot, rem = divmod(x, 10)
    while quot:
        result += rem
        quot, rem = divmod(quot, 10)
    return result + rem


if __name__ == "__main__" and len(sys.argv) > 1:
    with open(sys.argv[1], "r") as f:
        lines = f.readlines()
        testcases = ((int(x) for x in line.split(" ")) for line in lines[1:])

        def abc(a, b, c):
            """Massage input to requirements of problem."""
            return a * b + c

        for xs in testcases:
            n = abc(*xs)
            print(sum_digits(n))
```

## Tests

```py
from solution import sum_digits
from hypothesis import given
from hypothesis.strategies import integers


def test_sum_digits_mytest_simple():
    """My initial tests."""
    assert sum_digits(0) == 0, "expected sum 0 for value 0"
    assert sum_digits(1) == 1, "expected sum 1 for value 1"
    assert sum_digits(10) == 1, "expected sum 1 for value 10"
    assert sum_digits(123) == 6, "expected sum 6 for value 123"


def test_sum_digits_mytest_range():
    """My initial test for a range of integers."""
    numstr, total = "1", 1
    for i in range(2, 10):
        numstr += str(i)
        total += i
        assert (
            sum_digits(int(numstr)) == total
        ), f"expected sum {total} for value {numstr}"


def test_provided():
    """Tests provided with problem."""
    assert sum_digits(1492) == 16, "expected sum 16 for value 1492"
    assert sum_digits(1776) == 21, "expected sum 21 for value 1776"
    assert (
        sum_digits(11 * 9 + 1) == 1
    ), f"expected sum 1 for value \
            {11*9+1}"
    assert (
        sum_digits(14 * 90 + 232) == 16
    ), f"expected sum 16 for value \
            {14*90+232}"
    assert (
        sum_digits(111 * 15 + 111) == 21
    ), f"expected sum 21 for value \
            {111*15+111}"


def gen_int():
    """Configure generation for test_property_test()."""
    return integers(min_value=0, max_value=100)


@given(gen_int(), gen_int(), gen_int())
def test_property_test(a, b, c):
    """Property test using a range of integers."""
    n = a * b + c
    _sum = sum(int(x) for x in tuple(str(n)))
    assert sum_digits(n) == _sum, f"expected sum {_sum} for value n"
```
