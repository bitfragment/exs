---
title: "Fizz Buzz with range"
date: 2019-03-22
tags: javascript
---

## Source

Source: [CodeEval]: [Fizz Buzz]

[CodeEval]: https://www.codeeval.com/
[Fizz Buzz]: https://www.codeeval.com/open_challenges/1/

Given three positive integers *x*, *y*, *n*, in which
*x* is the first divisor, *y* is the second divisor, and *n* is a
maximum value, return an array of values 1 through *n* in which:

- numbers divisible by *x* are replaced with "F"
- numbers divisible by *y* are replaced with "B"
- numbers divisible by both *x* and *y* are replaced with "FB"


```
3, 5, 10 ⟹ [1, 2, "F", 4, "B", "F", 7, 8, "F", "B"]

2, 7, 15 ⟹ [1, "F", 3, "F", 5, "F", "B", "F", 9, "F", 11, "F", 13, "FB", 15]
```


## My solutions: without range

Imperative:

```js
const fizzbuzz = (x, y, n) => {
    let result = '';
    if (!(n % x)) result += 'F';
    if (!(n % y)) result += 'B';
    if (!result) result = n;
    return result;
};
```


Functional, using ternary conditional operators:

```js
const fizzbuzz = (x, y, n) =>
    !(n % x) && !(n % y)
        ? 'FB'
        : !(n % x)
            ? 'F'
            : !(n % y)
                ? 'B'
                : n;
```

Functional, using arrow functions (lambdas) and ternary 
conditional operators:

```js
const fizzbuzz = (x, y, n) => {
    const fizz = (x, y, n) => !(n % x) ? 'F' : '';
    const buzz = (x, y, n) => !(n % y) ? 'B' : '';
    const result = fizz(x, y, n) + buzz(x, y, n)
    return !result ? n : result;
}
```


## My solutions: including range

Imperative:

```js
const fizzbuzzRange = (x, y, n) => {
    let result = [];
    for (let i = 1; i <= n; i++) {
        result.push(fizzbuzz(x, y, i));
    }
    return result;
};
```

Functional, using array initialization:

```js
> Array.from({length: 5}, (v, i) => i);
[ 0, 1, 2, 3, 4 ]
> Array.from({length: 5}, (v, i) => i + 1);
[ 1, 2, 3, 4, 5 ]
```

```js
const fizzbuzzRange = (x, y, n) =>
    Array.from({length: n}, (v, i) => i + 1)
        .map(elt => fizzbuzz(x, y, elt));
```

---

```js
module.exports = {fizzbuzz, fizzbuzzRange};
```


## Tests

```js
const assert = require('assert')
    , fizzbuzz = require('./fizzbuzzRange').fizzbuzz
    , fizzbuzzRange = require('./fizzbuzzRange').fizzbuzzRange;

describe('#fizzbuzz()', function () {
    it('should return "F" when n is divisible by x only',
        function () {
            assert.equal(fizzbuzz(3, 5, 6), "F");
    });
    it('should return "B" when n is divisible by x only',
        function () {
            assert.equal(fizzbuzz(3, 5, 10), "B");
    });
    it('should return "FB" when n is divisible by both x and y',
        function () {
            assert.equal(fizzbuzz(3, 5, 15), 'FB');
    });
    it('should return n when not divisible by either x or y',
       function () {
            assert.equal(fizzbuzz(3, 5, 2), 2);
        });
});

describe('#fizzbuzzRange()', function () {
    const result1 = [1, 2, "F", 4, "B", "F", 7, 8, "F", "B"]
        , result2 = [1, "F", 3, "F", 5, "F", "B", "F", 9, "F", 11, "F", 13, "FB", 15];
    it('should return the correct result', function () {
        assert.deepEqual(fizzbuzzRange(3, 5, 10), result1);
        assert.deepEqual(fizzbuzzRange(2, 7, 15), result2);
    });
});
```
