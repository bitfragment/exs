---
title: "Largest element"
date: 2019-03-18
tags: javascript
---

## Source

Adrian Neumann, [Simple Programming Problems](https://adriann.github.io/programming_problems.html), Lists, Strings problem 1: "Write a function that returns the largest element in a list."


## Description

Define a generic function that, given an array, returns the
largest element it contains.

Treat numeric values literally: `99` is the largest element in the
array `[0, 1, 99]`. Measure the size of an associative array (hash map)
by the number of keys it contains. If two or more elements are the 
largest, return the first largest element. If passed data that is not 
an array, or if the array is empty, return null.

```
0 ⟹ null

[] ⟹ null

[null, undefined, 1] ⟹ 1

[0, 999, 23] ⟹ 999

['foo', 'bar'] ⟹ 'foo'

[null, undefined, 'foobar', 5, [1, 2, 3], {a: 1, b: 2}] ⟹ 'foobar'
```


## My solution

```js
const getSize = x =>
    !x // Handle null and undefined.
        ? 0
        : typeof x === 'number' 
            ? x
            : typeof x === 'object'
                ? Object.keys(x).length
                : x.length; // Handle everything else.


const largest = arr =>
    !arr || !arr.length
        ? null
        : arr.reduce((a, b) => getSize(a) >= getSize(b) ? a : b);


module.exports = {getSize, largest};
```


## Tests

```js
const assert = require('assert')
    , getSize = require('./largest.js').getSize
    , largest = require('./largest.js').largest;


// getSize()
// ---------
describe('#getSize()', () => {
    it('Should return 0 for null and undefined', () => {
        assert.strictEqual(getSize(null), 0);
        assert.strictEqual(getSize(undefined), 0);
    });
    it('Should return the same value if given a numeric value', () => {
        assert.strictEqual(getSize(3), 3);
    });
    it('Should return the value of the length property of a string', () => {
        assert.strictEqual(getSize('foo'), 3);
    });
    it('Should return the value of the length property of an array', () => {
        assert.strictEqual(getSize([1, 2, 3]), 3);
    });
    it('Should measure the size of an object by the number of keys ' +
       'it contains', () => {
        assert.strictEqual(getSize({a:1, b:2}), 2);
    });
});


// largest()
// ---------
describe('#largest()', () => {

    describe('boundaries', () => {
        it('Should return null if passed anything but an array', () => {
            assert.strictEqual(largest(null), null);
        });
        it('Should return null if array is empty', () => {
            assert.strictEqual(largest([]), null);
        });
        it('Should treat elements without a length as having length 0', () => {
            assert.strictEqual(largest([null, undefined, 1]), 1);
        });
    });

    describe('expected values', () => {
        it('Should return the largest number in a list of numbers', () => {
            assert.strictEqual(largest([0, 999, 23]), 999);
        });
        it('Should return the longest string in a list of strings', () => {
            const a = 'foo', b = 'barbaz', c = 'qux';
            assert.strictEqual(largest([a, b, c]), b);
        });
        it('Should return the longest list in a list of lists', () => {
            const a = [0]
                , b = [1, 2, 3]
                , c = [4, 5];
            assert.deepStrictEqual(largest([a, b, c]), b);
        });
        it('Should return the largest object in a list of objects', () => {
            const a = {x: 0}
                , b = {x: 0, y: 1, z: 2}
                , c = {x: 0, y: 1};
            assert.deepStrictEqual(largest([a, b, c]), b);
        });
    });

    describe('expected behavior', () => {
        it('Should return the first of two or more equal largest elements', () => {
            const a = [0]
                , b = [1, 2]
                , c = [4, 5];
            assert.deepStrictEqual(largest([a, b, c]), b);
        });
        it('Should correctly handle arrays of mixed data types', () => {
            let a = null
            , b = undefined
            , c = 5
            , d = 'foo'
            , e = [1, 2, 3, 4]
            , f = {a: 1, b: 2};
            assert.strictEqual(largest([a, b, c, d, e, f]), c);

            e = [1, 2, 3, 4, 5, 6];
            assert.strictEqual(largest([a, b, c, d, e, f]), e);

            d = 'foobarbaz';
            assert.strictEqual(largest([a, b, c, d, e, f]), d);        
        });
    });

});
```
