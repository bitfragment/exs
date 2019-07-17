---
title: String manipulation
date: 2019-07-17
tags: python
src-author: Edabit
src-title: "String Flips"
src-url: https://edabit.com/challenge/JsLu5qWsJtuJuBZT4
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

> Create a function that takes a string as the first argument, and a (string)
> specification as a second argument. If the specification is "word," return a
> string with each word reversed while maintaining their original order. If the
> specification is "sentence," reverse the order of the words in the string,
> while keeping the words intact.

## My solution

```py
def solution(data: str = "", spec: str = "") -> str:
    """Flip a string.

    If `spec` is "word," reverse characters in each word in `data`.
    If `spec` is "sentence," reverse the order of words in `data`.

    Args:
        data: A string to manipulate.
        spec: A specifier of a type of manipulation.

    Returns:
        A string representing a manipulation of the input string.

    Examples:
        >>> solution("Hello", "word")
        'olleH'

    """
    if not data or not str:
        return data
    if spec == "word":
        rev = lambda x: "".join(list(x)[::-1])
        words = [rev(word) for word in data.split()]
    if spec == "sentence":
        words = data.split()[::-1]
    return " ".join(words)
```

## Tests

```py
import pytest
from solution import solution


def test_ret_type():
    """Should return True for no argument."""
    assert isinstance(solution(), str), "expected True for no argument"


def test_default_args():
    """Should return empty string for fewer than two arguments."""
    assert solution() == "", "expected empty string for no argument"
    assert (
        solution("") == ""
    ), "expected empty string for empty string argument"


def test_empty_string_as_input():
    """Should reutrn empty string for empty string argument."""
    assert (
        solution("", "") == ""
    ), "expected empty string for empty string argument and empty spec argument"
    assert (
        solution("", "word") == ""
    ), "expected empty string for empty string argument and spec argument 'word'"
    assert (
        solution("", "sentence") == ""
    ), "expected empty string for empty string argument and spec argument 'sentence'"
    assert (
        solution("", "invalid spec") == ""
    ), "expected empty string for empty string argument and spec argument 'invalid spec'"


def test_word():
    """Should manipulate string according to 'word' specifier."""
    assert solution("Hello", "word") == "olleH"


def test_sentence():
    """Should manipulate string according to 'sentence' specifier."""
    assert solution("Hello world", "sentence") == "world Hello"


txt1 = "There's never enough time to do all the nothing you want"
txt2 = "I have all these great genes but they're recessive"
txt3 = "I like maxims that don't encourage behavior modification"


@pytest.mark.parametrize(
    "string, spec, expected",
    [
        (
            txt1,
            "word",
            "s'erehT reven hguone emit ot od lla eht gnihton uoy tnaw",
        ),
        (
            txt1,
            "sentence",
            "want you nothing the all do to time enough never There's",
        ),
        (txt2, "word", "I evah lla eseht taerg seneg tub er'yeht evissecer"),
        (
            txt2,
            "sentence",
            "recessive they're but genes great these all have I",
        ),
        (
            txt3,
            "word",
            "I ekil smixam taht t'nod egaruocne roivaheb noitacifidom",
        ),
        (
            txt3,
            "sentence",
            "modification behavior encourage don't that maxims like I",
        ),
    ],
)
def test_provided_tests(string, spec, expected):
    """Tests provided with problem."""
    assert (
        solution(string, spec) == expected
    ), f"expected {expected} for string argument {string} and spec argument {spec}"
```
