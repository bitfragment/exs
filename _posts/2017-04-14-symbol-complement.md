---
title: Symbol complement
date: 2017-04-14
tags: javascript
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

The task here is to swap symbols in a sequence with their defined complements.
One often finds this presented as a problem in e.g. "RNA transcription."

For this one needs a way to pair complementary data values and look up a
complementary value in either direction. The simple approach used here is to
pair an object containing key-value pairs with the same object "mirrored."

## My solution

### 1. Mirror an object

In the mirrored object, each value in the original object is now a key, and
each key in the original object is now a value.

```js
/**
 * Mirror an object.
 *
 * @param  {Object} obj - An object.
 * @return {Object} An object mirroring key-value pairs of the input object.
 *
 * @example
 * > mirror({'Z': 'A', 'Y': 'B'})
 * { A: 'Z', B: 'Y' }
 */
const mirror = obj =>
    Object.keys(obj).reduce((acc, curr) => {
        acc[obj[curr]] = curr;
        return acc;
    }, {});
```

### 2. Reversible look-up

Together, the original object and the mirrored object permit "reversible"
look-up via short-circuit evaluation.

```js
/**
 * Swap each symbol in a string with its complement.
 *
 * @param {string} str - An input string.
 * @param {Object} compls - An object defining symbols and their complements.
 * @return {string} The input string's complement.
 *
 * @example
 * > const compls = {'A': 'T', 'C': 'G'}
 * > strCompl('ATTGC', compls)
 * 'TAACG'
 */
const strCompl = (str, compls) =>
    [...str]
    .map(char => compls[char] || mirror(compls)[char])
    .join('');
```

## Tests

```js
const assert = require('assert');


describe('mirror()', function() {
    it("should return a correctly mirrored object", function () {
        const obj = {'A': 'Z', 'B': 'Y'};
        assert.deepEqual(mirror(obj), {'Z': 'A', 'Y': 'B'});
    });
});


describe('strCompl()', function () {
    it('should swap symbols with their complements', function () {
        const compls = {'A': 'T', 'C': 'G'};
        assert.equal(strCompl('ATTGC', compls), 'TAACG')
    });
});
```
