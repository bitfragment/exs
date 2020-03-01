---
title: Report duplicates
date: 2020-02-29
tags: python
src-author: CodeAbbey
src-title: Matching Words
src-url: https://www.codeabbey.com/index/task_view/matching-words
---


## Contents
{:.no_toc}


* TOC
{:toc}


## Description

> Input data consist of about 300 words, each made of 3 Latin letters.
> The end is signaled by the word `end`. Answer should contain all the
> words which are encountered more than once in lexicographic order.


## Solution sketches

### Solution sketch 1, using two lists

```py
# Data is unsorted, result must be sorted. Wait and sort
# the result, which will be smaller.
data = 'nun lam mip tex bal pif sot bal bod tex end'

scanned = []
multiples = []

for d in data.split(' '):
    if d not in scanned:
        scanned.append(d)
    elif d not in multiples:
        multiples.append(d)

assert ' '.join(sorted(multiples)) == 'bal tex'
```


### Solution sketch 2, using `count()`

```py
data = 'nun lam mip tex bal pif sot bal bod tex end'.split(' ')

multiples = []

for d in data:
    # Count only in this case.
    if d not in multiples and data.count(d) > 1:
        multiples.append(d)

assert ' '.join(sorted(multiples)) == 'bal tex'
```


### Solution sketch 3, using list comprehension

```py
data = 'nun lam mip tex bal pif sot bal bod tex end'.split(' ')

# Unnecessarily counts every time.
result = sorted(set([d for d in data if data.count(d) > 1]))

assert ' '.join(sorted(multiples)) == 'bal tex'

assert solution('') == ''
assert solution('a') == 'a'
assert solution('abc') == 'abc'

assert solution('nun lam mip tex bal pif sot bal bod tex end') == 'bal tex'
```


## Solutions

```py
from collections import Counter, defaultdict
from typing import Dict, List
```


### Solution 1

```py
def solution1(seq: str = "") -> str:
    """Solution using lists and `count`.

    Args:
        `seq`: A string representing a sequence of words.

    Returns:
        A string representing duplicated items in `seq`.

    Examples:
        >>> solution1('foo bar bar baz baz baz')
        'bar baz'

    """
    if not seq or " " not in seq:
        return seq
    multiples: List[str] = []
    items = seq.split(" ")
    for item in items:
        if item not in multiples and items.count(item) > 1:
            multiples.append(item)
    return " ".join(sorted(multiples))
```


### Solution 2

```py
def solution2(seq: str = "") -> str:
    """Solution using dictionary.

    Three ways to set the initial key value:
    1. counts[word] = 1 if word not in counts else counts[word] + 1
    2. counts[word] = counts.setdefault(word, 0) + 1
    3. Use `defaultdict`, as here.

    Args:
        `seq`: A string representing a sequence of words.

    Returns:
        A string representing duplicated items in `seq`.

    Examples:
        >>> solution2('foo bar bar baz baz baz')
        'bar baz'

    """
    if not seq or " " not in seq:
        return seq
    counts: Dict[str, int] = defaultdict(lambda: 0)
    for word in seq.split(" "):
        counts[word] += 1
    multiples = (x for x in counts if counts[x] > 1)
    return " ".join(sorted(multiples))
```


### Solution 3

```py
def solution3(seq: str = "") -> str:
    """Solution using `Counter`.

    Args:
        `seq`: A string representing a sequence of words.

    Returns:
        A string representing duplicated items in `seq`.

    Examples:
        >>> solution3('foo bar bar baz baz baz')
        'bar baz'

    """
    if not seq or " " not in seq:
        return seq
    counts = Counter(seq.split(" "))
    multiples = (x for x in counts if counts[x] > 1)
    return " ".join(sorted(multiples))
```



## Benchmarking

```shell
------------------------ benchmark: 3 tests ------------------------
Name (time in ms)                 Mean              Median
--------------------------------------------------------------------
test_benchmark[solution1]     262.7338 (1.0)      261.1003 (1.0)
test_benchmark[solution3]     288.3157 (1.10)     291.4873 (1.12)
test_benchmark[solution2]     344.2283 (1.31)     343.9028 (1.32)
--------------------------------------------------------------------
```


## Complexity

```shell
radon:
    F 73:0 solution1 - B
    F 99:0 solution2 - B
    F 129:0 solution3 - A
mccabe:
    73:0: 'solution1' 4
    99:0: 'solution2' 3
    129:0: 'solution3' 2
```


## Tests

```py
from random import randint
from string import ascii_letters

import pytest
from faker import Faker
from hypothesis import given
from hypothesis.strategies import sets, text

from solution import solution1, solution2, solution3
```

### Test fixtures

```py
@pytest.fixture(params=[solution1, solution2, solution3])
def get_solution_to_test(request):
    """Return the solution function to be tested."""
    yield request.param
```


### Test classes

```py
class TestArgumentAndReturn:
    """Test argument and return value."""

    @pytest.mark.xfail
    def test_unimplemented_input_validation(self, get_solution_to_test):
        """Should fail for unimplemented input validation."""
        f = get_solution_to_test
        assert not f(123)

    def test_return_datatype(self, get_solution_to_test):
        """Return value type should be `str`."""
        f = get_solution_to_test
        assert isinstance(f(), str)


class TestMeaninglessResults:
    """Test edge case inputs producing meaningless results."""

    def test_no_input(self, get_solution_to_test):
        """Should return empty string for no input."""
        f = get_solution_to_test
        assert not f()

    def test_empty_string(self, get_solution_to_test):
        """Should return empty string for empty string as input."""
        f = get_solution_to_test
        assert not f("")

    def test_single_char(self, get_solution_to_test):
        """Should return input string if it cannot be broken into words."""
        f = get_solution_to_test
        assert f("a") == "a"

    def test_single_word(self, get_solution_to_test):
        """Should return input string if it cannot be broken into words."""
        f = get_solution_to_test
        assert f("abc") == "abc"


class TestMeaningfulResults:
    """Test expected input producing meaningful results."""

    __words = ("foo", "bar", "baz")

    def generate_seq(self, begin, end, elts) -> str:
        """Generate a testable simple sequence.

        Args:
            `begin`: start (inclusive) of a range used to repeat elements
            `end`: end (exclusive) of a range used to repeat elements
            `elts`: the elements to repeat

        Returns:
            A string representing a testable simple sequence.

        Examples:
            >>> tc = TestMeaningfulResults()
            >>> tc.generate_seq(1, 4, ('foo', 'bar', 'baz'))
            'foo bar bar baz baz baz'

        """
        seqs = (i * f"{x} " for i, x in zip(range(begin, end), elts))
        return "".join(seqs).rstrip()

    def test_range_short(self, get_solution_to_test):
        """Should dete repetition of two of three elements."""
        f = get_solution_to_test
        data = self.generate_seq(1, 4, self.__words)
        assert f(data) == "bar baz"

    def test_range_short_rand(self, get_solution_to_test):
        """Should dete repetition when elements are randomly varied."""
        f = get_solution_to_test
        n = randint(1, 4)
        data = self.generate_seq(1, 4, self.__words)
        data = (n * "qux ") + data
        if n < 2:
            assert f(data) == "bar baz"
        else:
            assert f(data) == "bar baz qux"

    def test_range_long_1(self, get_solution_to_test):
        """Should dete repetition of three of four elements."""
        f = get_solution_to_test
        data = "qux " + self.generate_seq(100, 400, self.__words)
        result = f(data)
        assert len(result.split(" ")) == 3

    def test_range_long_2(self, get_solution_to_test):
        """Should dete repetition of all four elements."""
        f = get_solution_to_test
        data = "qux " + self.generate_seq(100, 400, self.__words)
        result = f("qux " + data)
        assert len(result.split(" ")) == 4


class TestProvidedTests:
    """Tests provided with problem."""

    def test_default_provided_test(self, get_solution_to_test):
        """Should detect repetition of two of eleven elements."""
        f = get_solution_to_test
        assert f("nun lam mip tex bal pif sot bal bod tex end") == "bal tex"

    def test_provided_longdata(self, get_solution_to_test):
        """Should detect repetition of 66 of 303 elements."""
        f = get_solution_to_test
        testdata = (
            "leq dof neq neh jat dux gox jyc nut max gac mak vef mys gic goq "
            "loc lek dak nis bip nyc bix juq gyp lec mef meq deq rok nif riq "
            "nux nok zus jat ryh mif ros gys mop jos gas duk ges lic naf lyf "
            "gys lec gux jus joh nyh byk mic zut ges jic ruf lap rup lux dap "
            "vyk lyq mos dix but roh jes bic boq vas vux net zet duc nih bop "
            "rak mus nos lih vys lof rat des raq mos got dux lih jyc gus gaq "
            "voh ref lys luh guf beh mox rax lep vok lap goc nif veh zyf jyk "
            "beh dup veh vyf zit jyh vef doq lah naf mit zix vek luf zet laq "
            "bak lyh dut jif doq zeh lyt lif zac dif myc nuc gak zes dot nof "
            "nuf naq dif byt ryf gas muq rux vyt ryh vyt let nok gic mot bys "
            "zis vaf res myp mac jah jys meh mek mip nac laq mup jup muh luf "
            "bys voh mix bic lyh nac not lik juq rap rap vaq nac moc vus lup "
            "nih zip mac vac jis gik jot dec gep nih jyp gos zac vys vep rip "
            "neq gux vak buc dyc voh myq gip joc bit ryc gis guf zak raf mok "
            "rat bah vuf jop byf vet las vyx nyp dek nac vyq dac mac net not "
            "gaf rok bak zas zih gap joc zik gyp lyk jys goq lyh net zut vis "
            "mak guf lok dof goc vyh mat jos vyp voq nis dut vis gyp lof nyf "
            "gip dut jus byk ruq nos gyk jes dox gof zek lak zyc raf beh lux "
            "bus res bak ryp gas lox nuc doh nyx jeq viq vit rap lyf end"
        )
        assert f(testdata) == (
            "bak beh bic byk bys dif dof doq dut dux gas ges gic gip goc "
            "goq guf gux gyp gys jat jes joc jos juq jus jyc jys lap laq lec "
            "lih lof luf lux lyf lyh mac mak mos nac naf neq net nif nih nis "
            "nok nos not nuc raf rap rat res rok ryh vef veh vis voh vys vyt "
            "zac zet zut"
        )


class TestFakerTests:
    """Using Faker: https://faker.readthedocs.io."""

    data = ""

    def test_set_of_strings(self, get_solution_to_test):
        """Should return empty string for a set of unique elements."""
        f = get_solution_to_test
        n = Faker().pyint()
        self.data = " ".join(Faker().pyset(n, True, "str"))
        assert not f(self.data)

    def test_set_plus_repeated_elt(self, get_solution_to_test):
        """Should detect repetition of one element."""
        f = get_solution_to_test
        self.data = "foo foo " + self.data
        assert f(self.data) == "foo"


class TestPropertyTests:
    """Using Hypothesis: https://hypothesis.readthedocs.io/."""

    __given_data = given(
        sets(text(alphabet=ascii_letters, min_size=1), min_size=2)
    )

    @__given_data
    def test_set_of_strings(self, get_solution_to_test, data):
        """Should return empty string for a set of unique elements."""
        f = get_solution_to_test
        assert not f(" ".join(data))

    @__given_data
    def test_set_plus_repeated_elt(self, get_solution_to_test, data):
        """Should detect repetition of one element."""
        f = get_solution_to_test
        assert f("foo foo " + " ".join(data)) == "foo"


class TestBenchmarkTests:
    """Using pytest-benchmark: https://pytest-benchmark.readthedocs.io/."""

    def test_benchmark(self, get_solution_to_test, benchmark):
        """Should detect repetition of 1 element out of 1000001 elements."""
        f = get_solution_to_test
        data = "foo " * 1_000_000 + "bar"
        result = benchmark(lambda: f(data))
        assert result == "foo"
```


### Test coverage

```shell
---------- coverage: platform darwin, python 3.7.3-final-0 -----------
Name          Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------
solution.py      26      0     18      0   100%
```
