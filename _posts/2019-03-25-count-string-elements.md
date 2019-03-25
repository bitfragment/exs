---
title: "Count string elements"
date: 2019-03-25
tags: javascript
---

## Source

Source: Adapted from Jeffrey Hu, [Python programming exercises], Question 13

[Python programming exercises]: https://github.com/zhiwehu/Python-programming-exercises


## Description

Count letters and digits.

Given a string, return an array containing counts of 
letters and digits, in the format `[ls, ds]`.

```
'hello world! 123' ⟹ [10, 3]
```


## My solution

```js
const isAlpha   = s => /^[a-zA-Z]+$/.test(s);

const isNumeric = s => /^[0-9]+$/.test(s);

const countLd = s => [
    [...s].filter(isAlpha).length,
    [...s].filter(isNumeric).length
];


module.exports = {countLd};
```


## Tests

```js
const assert = require('assert')
    , countLd = require('./countLd.js').countLd;

describe('countLd()', function() {
    it('should return the correct counts', function() {
        const s = 'hello world! 123';
        assert.deepEqual(countLd(s), [10, 3]);
    });
});
```
