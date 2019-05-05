---
title: "Numeronym"
date: 2019-05-05
tags: python
---

Source: [Problem - 71A - Codeforces]

[Problem - 71A - Codeforces]: https://codeforces.com/problemset/problem/71/A

* TOC
{:toc}


## My solution

```py
def solution(x: str, maxlen: int = 10) -> str:
    """Transform a string into a numeronym.

    If string `x` is more than `maxlen` characters long, transform
    it into a numeronym. Otherwise, return `x` in its original form.

    Args:
        x: the string to transform
        maxlen: the maximum length of a string to leave untransformed,
                default 10.

    Returns:
        The original string, or a numeronym based on the original string.

    Examples:
        >>> solution('abcdefghij')
        'abcdefghij'
        >>> solution('abcdefghijk')
        'a9k'

    """
    inner_len = len(x) - 2
    if inner_len <= maxlen - 2:
        return x
    first, middle, last = x[0], str(inner_len), x[-1]
    return first + middle + last
```


## Tests

```py
from hypothesis import given, strategies as st
from solution import solution

# Maximum length of non-transformed string.
maxlen = 10

msg_no_transform = (
    "Expected no transformation " "of string <= 10 characters long"
)

msg_transform = "Expected transformation of string > 10 characters long"


def test_mytest_no_transform():
    """My initial tests."""
    assert (
        solution("", maxlen) == ""
    ), "Expected no transformation of empty string"
    assert (
        solution("x", maxlen) == "x"
    ), "Expected no transformation of single character"


def test_mytest_range():
    """My initial tests."""
    x = ""
    # <= maxlen, so don't convert
    for _ in range(maxlen):
        x += "a"
        assert solution(x, maxlen) == x, msg_no_transform
    # > maxlen, so convert
    for i in range(maxlen, maxlen * 2):
        x += "a"
        first, middle, last = x[0], str(i - 1), x[-1]
        assert solution(x, maxlen) == first + middle + last, msg_transform


def test_provided_no_transform():
    """Tests provided with the problem."""
    assert solution("word", maxlen) == "word", msg_no_transform
    assert solution("abcdefghij", maxlen) == "abcdefghij", msg_no_transform


def test_provided_transform():
    """Tests provided with the problem."""
    assert solution("abcdefghijk", maxlen) == "a9k", msg_transform
    assert solution("localization", maxlen) == "l10n", msg_transform
    assert solution("internationalization", maxlen) == "i18n", msg_transform
    assert (
        solution("pneumonoultramicroscopicsilicovolcanoconiosis", maxlen)
        == "p43s"
    ), msg_transform


@given(
    st.text(
        alphabet=st.characters(min_codepoint=48, max_codepoint=122),
        max_size=10,
    )
)
def test_property_no_transform(x):
    """Property test for non-transformable data."""
    assert solution(x, maxlen) == x, msg_no_transform


@given(
    st.text(
        alphabet=st.characters(min_codepoint=48, max_codepoint=122),
        min_size=11,
    )
)
def test_property_transform(x):
    """Property test for transformable data."""
    first, middle, last = x[0], str(len(x) - 2), x[-1]
    assert solution(x, maxlen) == first + middle + last, msg_transform
```
