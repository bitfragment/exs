---
title: Sum digits
date: 2017-03-23
tags: python
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

Sum the digits of an integer.


## 1. sumdigits()

We can do this by manipulating numeric characters in a string,
a solution that stops at the sum of the digits in the input integer:  
1 + 2 + 3 + 4 + 5 + 6 + 7 = 28

```py
def sumdigits(n):
    """Return the sum of digits in the input integer treated as a string.

    Args:
        n (int): An integer.
    Returns:
        int: The sum of digits in the input integer treated as a string.

    Examples:
        >>> sumdigits(123)
        6
    """
    return sum(int(char) for char in str(n))
```


## 2. sumdigits_reduce()

Or we can do this mathematically, which reduces the input integer to
a single-digit sum:

1 + 2 + 3 + 4 + 5 + 6 + 7 = 28  
28 => 2 + 8 = 10  
10 => 1 + 0 = 1  

```py
def sumdigits_reduce(n):
    """Reduce values of digits in the input integer to a single-digit sum.

    Args:
        n (int): An integer.
    Returns:
        int: A single-digit sum of digits in the input integer.

    Examples:
        >>> sumdigits_reduce(1234567)
        1
    """
    return n if n < 10 else (n - 1) % 9 + 1
```


## 3. digitsums()

Let's collect the sums.

```py
def digitsums(n):
    """Return sums of digits beginning with the input integer.

    Args:
        n (int): An integer.
    Returns:
         int: An array of sums of digits.

    Examples:
        >>> list(digitsums(12345))
        [15, 6]
        >>> list(digitsums(99))
        [18, 9]
        >>> list(digitsums(100))
        [1]
    """
    s = sumdigits(n)
    yield s
    while s > 9:
        s = sumdigits(s)
        yield s
```


## 4. digitsums_count()

...or a count of sums.

```py
def digitsums_count(n):
    """Return a count of sums of digits beginning with the input integer.

    Args:
        n (int): An integer.
    Returns:
        int: A count of sums of digits.

    Examples:
        >>> digitsums_count(99)
        2
        >>> digitsums_count(100)
        1
    """
    return 0 if n < 10 else len(list(digitsums(n)))
```


## Tests

```py
def test_sumdigits():
    assert not sumdigits(0)
    assert sumdigits(1) == 1
    assert sumdigits(123) == 6
    assert sumdigits(12345) == 15
    assert sumdigits(1234567) == 28


def test_sumdigits_reduce():
    assert not sumdigits_reduce(0)
    assert sumdigits_reduce(1) == 1
    assert sumdigits_reduce(123) == 6
    assert sumdigits_reduce(12345) == 6
    assert sumdigits_reduce(1234567) == 1


def test_digitsums():
    assert list(digitsums(12345)) == [15, 6]


def test_digitsums_count():
    assert digitsums_count(5) == 0
    assert digitsums_count(100) == 1
    assert digitsums_count(91) == 2
```
