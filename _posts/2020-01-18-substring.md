---
title: Substring repeating character
date: 2020-01-18
tags: python
src-author: CheckiO
src-title: Long Repeat
src-url: https://py.checkio.org/en/mission/long-repeat/
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

> Here you should find the length of the longest substring that consists
> of the same letter. For example, line "aaabbcaaaa" contains four
> substrings with the same letters "aaa", "bb","c" and "aaaa". The last
> substring is the longest one, which makes it the answer.


## Solution sketch 1

```py
_str = 'foooooooooobaaaaaaaar'

curr, longest = 1, 1

for i, c in enumerate(_str):
    try:
        if c == _str[i + 1]:
            curr += 1
        else:
            if curr > longest:
                longest = curr
            curr = 1
    except IndexError:
        pass

assert longest == 10
```


## Solution sketch 2

```py
_str = 'foooooooooobaaaaaaaar'

curr, longest = 1, 1

for i, c in enumerate(_str):
    try:
        curr = curr + 1 if _str[i + 1] == c else 1
    except IndexError:
        pass
    longest = curr if curr > longest else longest

longest = 0 if not _str else longest

assert longest == 10
```


## My solution

O(n).

```py
def solution(s: str = "") -> int:
    """Return length of the longest repetition of a character.

    Given a string `s`, return the length of the substring representing
    the greatest contiguous repetition of a single character in `s`.

    Args:
        s: a string

    Returns:
        An integer representing the longest relevant substring.

    Examples:
        >>> solution('aaab')
        3
        >>> solution('abbb')
        3
        >>> solution('aabbbc')
        3

    """
    curr = 1
    longest = 0 if not s else 1
    for i, c in enumerate(s):
        try:
            curr = curr + 1 if s[i + 1] == c else 1
        except IndexError:
            pass
        longest = max(curr, longest)

    return longest
```

## Solution bytecode

```
$ python -i solution.py
>>> import dis; dis.dis(solution)
 66           0 LOAD_CONST               1 (1)
              2 STORE_FAST               1 (curr)

 67           4 LOAD_FAST                0 (s)
              6 POP_JUMP_IF_TRUE        12
              8 LOAD_CONST               2 (0)
             10 JUMP_FORWARD             2 (to 14)
        >>   12 LOAD_CONST               1 (1)
        >>   14 STORE_FAST               2 (longest)

 68          16 SETUP_LOOP              84 (to 102)
             18 LOAD_GLOBAL              0 (enumerate)
             20 LOAD_FAST                0 (s)
             22 CALL_FUNCTION            1
             24 GET_ITER
        >>   26 FOR_ITER                72 (to 100)
             28 UNPACK_SEQUENCE          2
             30 STORE_FAST               3 (i)
             32 STORE_FAST               4 (c)

 69          34 SETUP_EXCEPT            32 (to 68)

 70          36 LOAD_FAST                0 (s)
             38 LOAD_FAST                3 (i)
             40 LOAD_CONST               1 (1)
             42 BINARY_ADD
             44 BINARY_SUBSCR
             46 LOAD_FAST                4 (c)
             48 COMPARE_OP               2 (==)
             50 POP_JUMP_IF_FALSE       60
             52 LOAD_FAST                1 (curr)
             54 LOAD_CONST               1 (1)
             56 BINARY_ADD
             58 JUMP_FORWARD             2 (to 62)
        >>   60 LOAD_CONST               1 (1)
        >>   62 STORE_FAST               1 (curr)
             64 POP_BLOCK
             66 JUMP_FORWARD            20 (to 88)

 71     >>   68 DUP_TOP
             70 LOAD_GLOBAL              1 (IndexError)
             72 COMPARE_OP              10 (exception match)
             74 POP_JUMP_IF_FALSE       86
             76 POP_TOP
             78 POP_TOP
             80 POP_TOP

 72          82 POP_EXCEPT
             84 JUMP_FORWARD             2 (to 88)
        >>   86 END_FINALLY

 73     >>   88 LOAD_GLOBAL              2 (max)
             90 LOAD_FAST                1 (curr)
             92 LOAD_FAST                2 (longest)
             94 CALL_FUNCTION            2
             96 STORE_FAST               2 (longest)
             98 JUMP_ABSOLUTE           26
        >>  100 POP_BLOCK

 75     >>  102 LOAD_FAST                2 (longest)
            104 RETURN_VALUE
```


## Tests

```py
import pytest

from hypothesis import given
from hypothesis import strategies as st
from random import sample
from solution import solution


class TestArgumentAndReturn:
    """Test passed argument and return value."""

    @pytest.mark.xfail
    def test_wrong_arg_type(self):
        """Should fail for unimplemented argument data type."""
        assert (
            solution(1) == 0
        ), "expected failure for unimplemented input validation"

    def test_return_type(self):
        """Should return value of type integer."""
        assert isinstance(solution(), int), "expected return type to be int"


class TestMeaninglessResults:
    """Test edge cases producing meaningless results."""

    def test_no_input(self):
        """Should return 0 for no input."""
        assert solution() == 0, "expected 0 for no input"

    def test_empty_string_input(self):
        """Should return 0 for empty string."""
        assert solution("") == 0, "expected 0 for empty string"

    def test_single_char_input(self):
        """Should return 1 for one-character string."""
        assert solution("a") == 1, "expected 1 for one-character string"


class TestSimpleMeaningfulResults:
    """Test simplest possible meaningful results."""

    def test_simplest_meaningful_result(self):
        """Should return 1 for string with no repeated characters."""
        assert solution("ab") == 1, "expected 1 for string 'ab'"

    def test_shortest_long_string(self):
        """Should return 2 for a string repeating two characters."""
        assert solution("aa") == 2, "expected 2 for string 'aa'"


class TestPositional:
    """Place the longest substring in different positions in the string."""

    def test_longest_first(self):
        """Should return 3 for repetition of 3 at start of string."""
        assert solution("aaab") == 3, "expected 3 for string 'aaab'"

    def test_longest_last(self):
        """Should return 3 for repetition of 3 at end of string."""
        assert solution("abbb") == 3, "expected 3 for string 'abbb'"

    def test_longest_middle(self):
        """Should return 3 for repetition of 3 in middle of string."""
        assert solution("aabbbc") == 3, "expected 3 for string 'aabbbc'"


class TestRandomlyGenerated:
    """Test with randomly generated strings."""

    def test_randomly_generated_string(self):
        """Should correctly evaluate a randomly generated string."""
        strs = [x * str(x) for x in sample(range(10), 5)]
        longest = max([len(x) for x in strs])
        strs_concat = "".join(strs)
        assert longest == solution(
            strs_concat
        ), f"expected {longest} for string '{strs_concat}'"


class TestProvidedTests:
    """Tests provided with problem."""

    @pytest.mark.parametrize(
        "test_input, expected",
        [
            ("", 0),
            ("a", 1),
            ("aa", 2),
            ("aaba", 2),
            ("abababa", 1),
            ("abababaab", 2),
            ("sdsffffse", 4),
            ("ddvvrwwwrggg", 3),
        ],
    )
    def test_provided_tests(self, test_input, expected):
        """Should correctly evaluate provided test data."""
        assert (
            solution(test_input) == expected
        ), f"expected {expected} for string '{test_input}'"


@given(a=st.text("A"), b=st.text("B"))
def test_property_test_two_repeated_chars(a, b):
    """Should correctly evaluate automatically generated property test data."""
    longest = max(len(a), len(b))
    assert solution(a + b) == longest, f"expected {longest} for string {a + b}"


def _benchmark_setup():
    """Create benchmarking data."""
    s = "A" + "B" * 1000
    return (s,), {}


def _make_benchmark(solution):
    """Create a benchmarking function."""

    def f(benchmark):
        """Benchmark the solution."""
        result = benchmark.pedantic(solution, setup=_benchmark_setup)
        assert result == 1000, "expected 1000 (benchmarking test)"

    return f


test_benchmark_solution = _make_benchmark(solution)
```
