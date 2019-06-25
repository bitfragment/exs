---
title: Utilities
date: 2017-09-25
tags: javascript
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Array.prototype.indexOf()

Extend `Array.prototype.indexOf` to take a callback.

```js
> [ 1, 3, 5, 6, 7 ].indexOf(8)
-1
> [ 1, 3, 5, 6, 7 ].indexOf(v => v > 7)
-1
> [ 1, 3, 5, 6, 7 ].indexOf(1)
0
> [ 1, 3, 5, 6, 7, 1 ].indexOf(1, 2)
5
> [ 1, 3, 5, 6, 7, 8 ].indexOf(v => v % 2 === 0)
3
> [ 1, 3, 5, 6, 7, 8 ].indexOf(v => v % 2 === 0, 4)
5
```

---

```js
Array.prototype.indexOf = function(x, fromIndex=0) {
    const _x = this.slice(fromIndex);
    for (let idx in _x) {
        idx = parseInt(idx, 10);
        const val = _x[idx];
        const found = idx + fromIndex;
        if ((typeof(x) === 'function' && x(val)) || val === x) {
            return found;
        }
    }
    return -1;
};
```

## chunkarr()

Chunk an array.

```js
> chunkArr([1, 2, 3, 4, 5, 6], 3)
[ [ 1, 2, 3 ], [ 4, 5, 6 ] ]
```

---

```js
const chunkArr = function(arr, size) {
    chunked = [];
    while (arr.length > 0) {
        chunked.push(arr.splice(0, size));
    }
    return chunked;
};
```

## cmpArr()

Compare arrays whose elements are primitive values.

```js
> cmpArr([1, 2, 3], [1, 2])
false
> cmpArr([1, 2, 3], [3, 2, 1])
false
> cmpArr([1, 2, 3], [1, 2, 3])
true
```

---

```js
const cmpArr = function(arr1, arr2) {
    if (arr1.length !== arr2.length) return false;
    for (let idx in arr1) {
        if (arr1[idx] !== arr2[idx]) {
            return false;
        }
    }
    return true;
};
```

## cmpStr()

Compare two string property values case-insensitively.

```js
> cmpStr({'a': 'b'}, {'a': 'c'}, 'a')
-1
> cmpStr({'a': 'b'}, {'a': 'B'}, 'a')
0
```

---

```js
const cmpStr = (obj1, obj2, val) =>
    obj1[val].toLowerCase().localeCompare(obj2[val].toLowerCase());
```

## count()

Count occurrences of `elt` in `seq`.

```js
> count(['a', 'a', 'b', 'c'], 'a')
2
> count('foobar', 'o')
2
```

`elt` can also be a function:

```js
> count(['a', 'a', 'b', 'c'], x => x === 'a')
2
```

---

```js
const count = function(seq, elt) {
    const s = typeof seq === 'string' ? [...seq] : seq.slice();
    return s.filter(x =>
        typeof elt === 'function' ? elt(x) : x === elt
    ).length
};
```

## domainName()

Return domain name.

```js
> domainName("http://google.com")
'google'
> domainName("http://google.co.jp")
'google'
> domainName("www.xakep.ru")
'xakep'
> domainName("xakep.ru")
'xakep'
> domainName("http://www.xakep.ru")
'xakep'
```

---

```js
const domainName = s =>
    s.replace(/http[s]*:\/\//, '')
    .replace('www.', '')
    .split(/\/\/|\./)[0];
```


## eltFreq()


Return an object holding frequency counts of array elements.

```js
> eltFreq(['a', 'a', 'b', 'c'])
{ 'a': 2, 'b': 1, 'c': 1 }
```

---

```js
const eltFreq = arr =>
    arr.reduce((freq, n) => {
        freq[n] = freq[n] + 1 || 1;
        return freq;
    }, {});
```

## fan()

'Fan' or unfurl characters in a sequence.

```js
> fan('foo') // string
[ 'f', 'fo', 'foo' ]
//
> fan(123)  // number
[ '1', '12', '123' ]
```

---

```js
const fan = x =>
    [...String(x)].map((curr, idx) => String(x).slice(0, idx + 1));
```

## idxs()

Extract characters at specified index positions and construct a
new sequence. If index is out of range, use empty string.

```js
> idxs([1, 2, 6, 7], 'foobarbarous')
'ooba'
```

---

```js
const idxs = (arr, s) =>
    arr.map(elt => s[elt] || '').join('');
```

## includesSeq()

Return true if `arr` contains sequence `seq`.
Expects `seq` to be at least two characters.

```js
const includesSeq = (arr, seq) =>
    arr.length &&
    2 <= seq.length &&
    seq.reduce((prev, curr) => prev &&
        arr.indexOf(curr) > arr.indexOf(prev));
```

## innerSort()

Sort by a primary element and then by a secondary element.

```js
> const data = [{'a': 'y', 'b': 'x'}, {'a': 'x', 'b': 'z'}, {'a': 'x', 'b': 'y'}]
> const compare = (a, b, x) => a[x].localeCompare(b[x])
> innerSort(data, compare, 'a', 'b')
[{a: 'x', b: 'y'}, {a: 'x', b: 'z'}, {a: 'y', b: 'x'}]
```

---

```js
const innerSort = (arr, f, x, y) =>
    arr.sort((a, b) => f(a, b, x))
       .sort((a, b) => f(a, b, x) ||
                       f(a, b, y));
```

## isVowelAscEng()

Return true if `c` is an ASCII English vowel character.

```js
> isVowelAscEng('a')
true
> isVowelAscEng('b')
false
```

---

```js
const vowelAscEng = "aeiouAEIOU";

const isVowelAscEng = c => !!c && vowelAscEng.includes(c);
```

## last()

Return last element of `arr`.

```js
> last([[0, 1], [1, 2], [3, 4]])
[ [ 3, 4 ] ]
```

---

```js
const last = arr => arr.slice(-1);
```

## listify()

Listify array of strings, including optional serial comma.

```js
> listify('a')
a
> listify(['a', 'b', 'c'], true)
a, b, and c
> listify(['a', 'b', 'c'], false)
a, b and c
```

---

```js
const listify = (arr, serial) =>
    arr.length === 1
        ? arr[0]
        : arr.slice(0, -1)
            .join(', ') +
            (serial ? ',' : '') +
            ` and ${arr.slice(-1)}`;
```

## mirror()

Mirror an object.

```js
> mirror({'Z': 'A', 'Y': 'B'})
{ A: 'Z', B: 'Y' }
```

---

```js
const mirror = obj =>
    Object.keys(obj).reduce((acc, curr) => {
        acc[obj[curr]] = curr;
        return acc;
    }, {});
```

## mults()

Yield multiples of `n` up to `max`, non-inclusive.

```js
> [...mul(2, 9)]
[ 2, 4, 6, 8 ]
```

---

```js
function* mults(n, max) {
    let i = 1, _next = n;
    while (_next < max) {
        yield _next;
        _next = n * ++i;
    }
}
```

## nextMinGreater()

Get the next smallest greater number following the
number at `arr[idx]`.

```js
> nextMinGreater([1, 2, 1, 3, 2], 2)
2
```

---

```js
const nextMinGreater = function(arr, idx) {
    let rest = arr.slice(idx + 1);
    rest = rest.filter(elt => elt > arr[idx]);
    return rest.length ? Math.min(...rest) : -1;
}
```

## nonConsec()

Return an array of numeric values in `arr` that do not follow
consecutively from preceding numeric values.

```js
> nonConsec([1, 2, 3, 4, 6, 7, 9])
[ 6, 9 ]
```

---

```js
const nonConsec = arr =>
    arr.filter((elt, idx, arr) =>
        elt - arr[idx - 1] > 1);
```

## nonUniq()

Return all non-unique values of `arr`.

```js
> nonUniq(['a', 'a', 'b', 'b', 'c'])
[ 'a', 'a', 'b', 'b' ]
```

---

```js
const nonUniq = arr => arr.filter(elt => count(arr, elt) > 1);
```

## nth()

Return `n`th element of `arr` (zero-indexed).

```js
> nth(['a', 'b', 'c'], 1)
[ 'b' ]
```

---

```js
const nth = (arr, n) =>
    n === -1
        ? last(arr)
        : arr.slice(n, n + 1);
```

## obj2arr()

Return an array corresponding to frequency of occurrence
tabulated in the input object.

```js
> obj2arr({'a': 2, 'b': 3})
[ 'a', 'a', 'b', 'b', 'b' ]
```

---

```js
const obj2arr = obj =>
    Object.keys(obj)
        .reduce((acc, key) => {
            acc.push(...Array.from(Array(obj[key]))
                .map(elt => (/\d/).test(key) ? parseInt(key) : key));
            return acc;
    }, []);
```

## Object.values()

Polyfill for [ES2017 draft Object.values()](https://github.com/tc39/proposal-object-values-entries).

Return an array containing the values in `obj`.

```js
> Object.values({'a': 1, 'b': 2, 'c': 3})
[ 1, 2, 3 ]
```

---

```js
Object.prototype.values = function(obj) {
    const result = [];
    for (const k of Object.keys(obj)) {
        result.push(obj[k]);
    }
    return result;
};
```

## randInt()

Return pseudo-random integer in range `min`, `max` (non-inclusive).

```js
> randInt(0,3)
0
> randInt(0,3)
2
```

---

```js
const randInt = (min, max) => Math.floor(Math.random() * max) + min;
```

## range()

Return range from `begin` to `end` (inclusive).

```js
> Array.from(range(0, 3))
[ 0, 1, 2, 3 ]
> Array.from(range(0, 6, 2))
[ 0, 2, 4, 6 ]
> Array.from(range(3, 0))
[ 3, 2, 1, 0 ]
> Array.from(range(6, 0, 2))
[ 6, 4, 2, 0 ]
```

---

```js
function* range(begin, end, step=1) {
    if (begin <= end) {
        while (begin <= end) {
            yield begin;
            begin += step;
        }
    } else {
        while (begin >= end) {
            yield begin;
            begin -= step;
        }
    }
}
```

## rangeBitCount()

Count occurrences of `1` in binary representations of all integers
in range `a` to `b` inclusive.

```js
> rangeBitCount(2, 7)
11
```

---

```js
const rangeBitCount = function(a, b) {
    let r = [];
    let g = range(a, ++b);
    while (b--) r.push(g.next().value);
    let s = r.map(elt => elt ? elt.toString(2) : '').join('');
    return count(s.split(''), '1');
};
```

## reversed()

Given a string `s`, return a string with the characters in `s`
reversed. Given an array `a`, return an array with the elements
in `a` reversed. Given a number `n`, return a number with the
digits in `n` reversed.

```js
> reversed('foobar')
'raboof'
> reversed(32243)
34223
> reversed(['a', 'b', 'c'])
[ 'c', 'b', 'a' ]
```

---

```js
const reversed = x =>
    typeof x === 'number'
        ? Number([...x.toString()].reverse().join(''))
        : typeof x === 'string'
            ? [...x].reverse().join('')
            : x.reverse();
```

## runningAverage()

Return a running average.

```js
> const rAvg = runningAverage()
undefined
> rAvg(1)
1
> rAvg(5)
3
> rAvg(10)
5.33
```

---

```js
const runningAverage = () => {
    let count = total = 0;
    return n => {
        total += n;
        count++;
        return Number((total / count).toFixed(2));
    };
};
```

## shiftNmg()

Replace every element of `arr` with the next smallest
greater number following it, or `-1` if none is found.

```js
> shiftNmg([1, 2, 3])
[ 2, 3, -1 ]
```

---

```js
const shiftNmg = arr =>
    arr.map((elt, idx) =>
        nextMinGreater(arr, idx));
```

## shuffle()

Shuffle an array.
Does not compare `arr` to return value, so no guarantees.

```js
> shuffle([1, 2, 3, 4, 5])
[ 2, 1, 3, 4, 5 ]
```

---

```js
const shuffle = function(arr) {
    let _arr = arr.slice();
    return arr.map(elt => _arr.splice(randInt(0, _arr.length), 1)[0]);
};
```

## shuffleInner()

Shuffle inner array (elements at indexes 1 through -2).

```js
> shuffleInner([1, 2, 3, 4, 5])
[ 1, 4, 2, 3, 5 ]
```

---

```js
const shuffleInner = arr =>
    arr.slice(0, 1)
    .concat(shuffle(arr.slice(1, -1)))
    .concat(arr.slice(-1));
```

## smash()

Smash all whitespace.

```js
> smash(' foo bar b a z ')
'foobarbaz'
```

---

```js
const smash = s =>
    s.replace(/\s/g, '');
```

## sortedn()

Sort an array of numeric values.

`Array.prototype.sort()` uses Unicode point order:

```js
> [1, 10, 21, 2].sort()
[ 1, 10, 2, 21 ] // <= oops
```

Compare:

```js
> sortedn([1, 10, 21, 2])
[ 1, 2, 10, 21 ]
```

---

```js
const sortedn = arr =>
    arr.sort((a, b) => a - b);
```

## strCompl()

Swap each symbol in a string with its complement.

```js
> const compls = {'A': 'T', 'C': 'G'}
> strCompl('ATTGC', compls)
'TAACG'
```

---

```js
const strCompl = (str, compls) =>
    [...str]
    .map(char => compls[char] || mirror(compls)[char])
    .join('');
```

## sum()

Sum arguments or array.

```js
> sum(1, 2, 3)
6
> sum([1, 2, 3])
6
```

---

```js
const sum = function(seq) {
    const add = (a, b) => a + b;
    return Array.isArray(seq)
        ? seq.reduce(add)
        : [...arguments].reduce(add);
};
```

## toUpperSubs()

Convert a substring to uppercase.

```js
> toUpperSubs('foobar', 2, 4)
'foOBar'
```

---

```js
const toUpperSubs = (s, begin, end) =>
    s.substring(0, begin) +
    s.substring(begin, end).toUpperCase() +
    s.substring(end);
```

## uniq()

Return non-unique element of `arr`.  
Takes optional function `proc`.  
Assumes only one non-unique element.  

```js
> uniq(['a', 'a', 'b', 'c', 'd'])
'b'
```

Some helper functions:

```js
> const smash = s => s.replace(/\s/g, '')
> smash('Foo Bar Baz')
'FooBarBaz'

> const setify = s => new Set([...smash(s).toLowerCase()])
> setify('Foo Bar Baz')
Set { 'f', 'o', 'b', 'a', 'r', 'z' }

> const proc = s => Array.from(setify(s)).sort().join('')
> proc('foo bar baz')
'abforz'
```

Invoke with function:

```js
> uniq(['Anagram',
        'Internet Anagram Server',
        'I Rearrangement Servant'
       ], proc)
'Anagram'
```

---

```js
const uniq = (arr, proc = x => x) => {
    let nonUniq = arr[0];
    for (let elt of arr.slice(1)) {
        if (proc(nonUniq) === proc(elt)) break;
        else nonUniq = elt;
    }
    return arr.filter(elt => proc(elt) !== proc(nonUniq))[0];
};
```

## ys()

Return y-values of an array of `[x, y]` arrays.

```js
> ys([[0, 1], [1, 2], [2, 3]])
[ 1, 2, 3 ]
```

---

```js
const ys = arr => arr.map(xy => xy[1]);
```
