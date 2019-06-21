---
title: Inner sort
date: 2017-04-15
tags: js, core
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

A common task is to sort items by a first element, then by a
second element, so that the collection is ordered by the
first element, but within any groups of that first element,
additionally ordered by the second.

For example, you may want to sort a collection of addresses
by country first (a primary or "outer" sort), then by city
(a secondary or "inner" sort).

## My solution

```js
/**
 * Sort by a primary element and then by a secondary element.
 *
 * @param {object} arr An unsorted array
 * @param {function} f A function that performs a comparison
 * @param {string} x The outer sort element
 * @param {string} y The inner sort element
 * @return {object} A sorted array
 *
 * @example
 * > const data = [{'a': 'y', 'b': 'x'}, {'a': 'x', 'b': 'z'}, {'a': 'x', 'b': 'y'}]
 * > const compare = (a, b, x) => a[x].localeCompare(b[x])
 * > innerSort(data, compare, 'a', 'b')
 * [{a: 'x', b: 'y'}, {a: 'x', b: 'z'}, {a: 'y', b: 'x'}]
 */
const innerSort = (arr, f, x, y) =>
    arr.sort((a, b) => f(a, b, x))
       .sort((a, b) => f(a, b, x) ||
                       f(a, b, y));
```

## A compare function

A function like `innerSort()` could be used with different compare
functions. Let's try a compare function that compares two object
property values that are strings.

```js
/**
 * Compare two string property values case-insensitively.
 *
 * @param {object} obj1 The first object to compare
 * @param {object} obj2 The second object to compare
 * @param {string} val A property name present in both objects
 * @return {number} The value returned by String.prototype.localeCompare()
 *
 * @example
 * > compare({'a': 'b'}, {'a': 'c'}, 'a')
 * -1
 * @example
 * > compare({'a': 'b'}, {'a': 'B'}, 'a')
 * 0
 */
const compare = (obj1, obj2, val) =>
    obj1[val].toLowerCase().localeCompare(obj2[val].toLowerCase());
```


## Tests

```js
const assert = require('assert');


describe('compare', () => {
    it('should return the value returned by ' +
       'String.prototype.localeCompare()', () => {
        assert.equal(-1, compare({'a': 'b'}, {'a': 'c'}, 'a'));
    });
});


describe('innerSort', () => {
    it('should sort an array of objects', () => {
        const data = [
            {'a': 'y', 'b': 'x'},
            {'a': 'x', 'b': 'z'},
            {'a': 'x', 'b': 'y'}
        ];
        const sorted = innerSort(data, compare, 'a', 'b');
        assert.deepEqual(sorted, [
            {'a': 'x', 'b': 'y'},
            {'a': 'x', 'b': 'z'},
            {'a': 'y', 'b': 'x'}
        ]);
    });
});
```
