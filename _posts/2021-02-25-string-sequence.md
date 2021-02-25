---
title: Evaluate string sequence
date: 2021-02-25
tags: python
src-author: Py.CheckiO
src-title: Goes Right After
src-url: https://py.checkio.org/en/mission/goes-after/
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

> In a given word you need to check if one symbol goes right after another. Cases you should expect while solving this challenge:
> 
> * If more than one symbol is in the list you should always count the first one;
> * One of the symbols are not in the given word - your function should return False;
> * Any symbol appears in a word more than once - use only the first one;
> * Two symbols are the same - your function should return False;
> * The condition is case sensitive, which means 'a' and 'A' are two different symbols.
> * Input: Three arguments. The first one is a given string, second is a symbol that should go first, and the third is a symbold that should go after the first one.
> * Output: A bool.

## Solution

```py
def solution1(s: str, sym_a: str, sym_b: str) -> bool:
    """Determine if `sym_b` immediately follows `sym_a` in `s`.

    Return `True` if `sym_b` follows the first occurrence of `sym_a`
    in `s`.

    Args:
        `s`: A string.
        `sym_a`: A string one character in length
        `sym_b`: A string one character in length

    Returns:
        A Boolean value

    Examples:
        >>> solution1('foobar', 'f', 'o')
        True
        >>> solution1('foobar', 'o', 'b')
        False

    """
    if sym_a == sym_b:
        return False
    try:
        sym_a_idx = s.index(sym_a)
        nxt_idx = sym_a_idx + 1
        nextchr = s[nxt_idx]
        return nextchr == sym_b and nxt_idx == s.index(nextchr)
    except (IndexError, ValueError):
        return False


def solution2(s: str, sym_a: str, sym_b: str) -> bool:
    """Do better than over-engineered `solution1`()`.

    26 lines in byte code, where `solution1()` was 46 lines.

    Examples:
        >>> solution1('foobar', 'f', 'o')
        True
        >>> solution1('foobar', 'o', 'b')
        False

    """
    try:
        return s.index(sym_b) == s.index(sym_a) + 1
    except ValueError:
        return False
```

### Benchmarking

```sh
----------------------- benchmark: 2 tests -----------------------
Name (time in us)                Mean             Median
------------------------------------------------------------------
test_benchmark[solution2]     40.1191 (1.01)     38.8900 (1.05)
test_benchmark[solution1]     39.6621 (1.0)      37.1960 (1.0)
------------------------------------------------------------------
```

### Complexity
```sh
radon:
./solution.py
    F 5:0 solution1 - A
    F 36:0 solution2 - A
mccabe:
5:0: 'solution1' 4
36:0: 'solution2' 3
```

## Tests

```py
import logging
import pytest

from faker import Faker
from hypothesis import given
from hypothesis.strategies import permutations
from random import choice, randint
from string import ascii_lowercase, ascii_uppercase
from typing import List, Tuple

from solution import solution1, solution2

logging.basicConfig(level=logging.DEBUG)
log = logging.getLogger(__name__)
```

### Test fixtures

```py
@pytest.fixture(scope="module", params=[solution1, solution2])
def get_function_to_test(request):
    """Designate the solution to test."""
    yield request.param
```


### Test classes

```py
class TestArgumentAndReturn:
    """Test arguments and return value."""

    def test_missing_arguments(self, get_function_to_test):
        """Should raise `TypeError` if required arguments are missing."""
        f = get_function_to_test
        with pytest.raises(TypeError):
            assert not f()
            assert not f("abc")
            assert not f("abc", "a")

    def test_unimplemented_input_validation(self, get_function_to_test):
        """Test for unimplemented input validation.

        Should raise `AttributeError` if first argument is not indexable.
        """
        f = get_function_to_test
        with pytest.raises(AttributeError):
            assert not f(123, 4, 5)

    def test_return_datatype(self, get_function_to_test):
        """Return value type should be `bool`."""
        f = get_function_to_test
        assert isinstance(f("a", "b", "c"), bool)


class TestMeaninglessResults:
    """Test edge case inputs producing meaningless results."""

    def test_truncated_input(self, get_function_to_test):
        """Should return False if length of first arg is < 2 chars."""
        f = get_function_to_test
        assert not f("a", "b", "c")


class TestMeaningfulResults:
    """Test expected input producing meaningful results."""

    def test_identical_symbols(self, get_function_to_test):
        """Should return False if second and third args are identical."""
        f = get_function_to_test
        assert not f("abc", "a", "a")

    def test_symbols_not_in_string(self, get_function_to_test):
        """Should correctly evaluate missing symbols.

        Should return False if second and third args are not found
        in first arg.
        """
        f = get_function_to_test
        assert not f("aa", "a", "x")
        assert not f("aa", "x", "a")
        assert not f("aa", "x", "x")

    def test_symbols_out_of_order(self, get_function_to_test):
        """Should correctly evaluate symbols out of order.

        Should return False if second and third args are found in first
        arg, but not in the desired order.
        """
        f = get_function_to_test
        assert not f("abcde", "c", "a")
        assert not f("abcde", "e", "a")

    def test_case_sensitivity(self, get_function_to_test):
        """Detection of second and third args should be case sensitive."""
        f = get_function_to_test
        assert f("abc", "a", "b")
        assert not f("abc", "A", "b")
        assert not f("abc", "a", "B")


class TestProvidedTests:
    """Tests provided with problem."""

    @pytest.mark.parametrize(
        "s, sym_a, sym_b, expected",
        [
            ("", "l", "o", False),
            ("almaz", "a", "l", True),
            ("almaz", "m", "a", False),
            ("almaz", "p", "p", False),
            ("almaz", "r", "a", False),
            ("almaz", "r", "l", False),
            ("list", "l", "l", False),
            ("list", "l", "o", False),
            ("panorama", "a", "n", True),
            ("world", "d", "w", False),
            ("world", "l", "o", False),
            ("world", "w", "o", True),
            ("world", "w", "r", False),
        ],
    )
    def test_provided_tests(
        self, get_function_to_test, s, sym_a, sym_b, expected
    ):
        """Should correctly evaluate provided test data."""
        f = get_function_to_test
        assert f(s, sym_a, sym_b) == expected


class SequenceHelpers:
    """Helper functions for use with generated sequences."""

    def __choose(self, excl: Tuple[str]) -> str:
        """Return a randomly chosen character.

        If that character is one a designated excluded character, return an empty string.
        """
        c = choice(ascii_lowercase)
        return c if c not in excl else ""

    def __middle_two(self, s: str) -> str:
        """Return a subsequence of two chars from the input string."""
        mid: int = len(s) // 2
        return s[mid: mid + 2]

    def gen_excl(self) -> Tuple[str]:
        """Return a tuple of randomly chosen characters."""
        a = choice(ascii_lowercase)
        b = choice(ascii_lowercase)
        while b == a:
            b = choice(ascii_lowercase)
        return (a, b)

    def seq(self, start: int, end: int, excl: Tuple[str]) -> str:
        """Return a sequence of randomly chosen characters."""
        return "".join(
            SequenceHelpers.__choose(self, excl)
            for x in range(randint(start, end))
        )

    def seq_pass(self, data: List[str]) -> Tuple[List[str], Tuple[str]]:
        """Return a passing sequence with excluded characters."""
        seq = SequenceHelpers.__middle_two(self, data)
        excl = tuple(seq)
        log.info(f"with passing sequence '{seq}' in '{data}'")
        return data, excl

    def seq_fail(self, data: List[str]) -> Tuple[List[str], Tuple[str]]:
        """Return a failing sequence with excluded characters."""
        data = data[0] + data
        seq = data[1:3]
        excl = tuple(seq)
        log.info(f"with failing sequence '{seq}' in '{data}'")
        return data, excl


class TestGeneratedSequences:
    """Tests of generated sequences."""

    def __seq(self, begin: int, end: int, excl: Tuple[str]) -> str:
        """Construct a testable sequence."""
        return SequenceHelpers.seq(self, begin, end, excl)

    @pytest.mark.parametrize("begin, end", [(5, 10), (10, 12), (12, 15)])
    def test_seqs(self, get_function_to_test, begin, end):
        """Should correctly evaluate randomly generated sequences."""

        f = get_function_to_test
        excl = SequenceHelpers.gen_excl(self)

        mystr = (
            self.__seq(begin, end, excl)
            + "".join(excl)
            + self.__seq(begin, end, excl)
        )
        log.info(f"with passing sequence '{''.join(excl)}' in '{mystr}'")
        assert f(mystr, *excl)

        excl = (choice(ascii_lowercase), choice(ascii_uppercase))
        log.info(f"with failing sequence '{''.join(excl)}' in '{mystr}'")
        assert not f(mystr, *excl)


class TestFakerTests:
    """Tests of sequences generated using Faker.

    See https://faker.readthedocs.io.
    """

    kwargs = {
        "nb_elements": Faker().pyint(5, 15),
        "variable_nb_elements": True,
        "value_types": "str",
    }
    ss = Faker().pyset(**kwargs)
    data: List[str] = ["".join(set(list(x))) for x in ss]

    def test_seqs(self, get_function_to_test):
        """Should correctly evaluate generated sequences."""

        f = get_function_to_test

        for _ in self.data:
            s, excl = SequenceHelpers.seq_pass(self, _)
            assert f(s, *excl)

            s, excl = SequenceHelpers.seq_fail(self, _)
            assert not f(s, *excl)


class TestPropertyTests:
    """Tests of sequences generated using Hypothesis.

    See https://hypothesis.readthedocs.io/.
    """

    __given_data = given(permutations(ascii_lowercase))

    @__given_data
    def test_seqs(self, get_function_to_test, data):
        """Should correctly evaluate generated sequences."""

        f = get_function_to_test
        data = "".join(data)

        s, excl = SequenceHelpers.seq_pass(self, data)
        assert f(s, *excl)

        s, excl = SequenceHelpers.seq_fail(self, data)
        assert not f(s, *excl)


class TestBenchmarkTests:
    """Tests for use with pytest-benchmark.

    See https://pytest-benchmark.readthedocs.io/.
    """

    def test_benchmark(self, get_function_to_test, benchmark):
        """Should correctly evaluate a generated sequence."""
        f = get_function_to_test
        s = "a" * 1_000_000 + "bc"
        result = benchmark(lambda: f(s, "b", "c"))
        assert result
```

### Test coverage

```sh
---- coverage: platform darwin, python 3.8.5-final-0 ----
Name          Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------
solution.py      15      0      2      0   100%
---------------------------------------------------------
TOTAL            15      0      2      0   100%
```
