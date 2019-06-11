---
title: MiniStringFuck interpreter
date: 2017-04-15
tags: javascript, core
---

## Contents
{:.no_toc}

* TOC
{:toc}

Implementation of a [MiniStringFuck] interpreter.

[MiniStringFuck]: http://esolangs.org/wiki/MiniStringFuck


## 1. Utilities

```js
/**
 * Increment value of a memory cell, resetting value to 0 when it
 * reaches a limit.
 *
 * @param  {number} mem An integer.
 * @param  {number} lim An integer
 * @return {number}     An integer.
 *
 * @example
 * > msfIncr(1, 256)
 * 2
 *
 * @example
 * > msfIncr(255, 256)
 * 0
 */
const msfIncr = (mem, lim) => (mem + 1) % lim;
```


## 2. Evaluation

```js
/**
 * Evaluate a character:
 * `+` increments memory;
 * `.` adds extended ASCII value of memory to output.
 *
 * @param {object} data An object containing memory and data.
 * @param {string} chr  A character.
 *
 * @example
 * > const data = { mem: 0, out: '' }
 * > let i = 65; while (i--) msfEval(data, '+')
 * > msfEval(data, '.')
 * > data.mem
 * 65
 * > data.out
 * A
 */
const msfEval = function(data, chr) {
    if (chr === '+') data.mem = msfIncr(data.mem, 256);
    else if (chr === '.') data.out += String.fromCharCode(data.mem);
};
```


## 3. Interpretation

```js
/**
 * Interpret a MiniStringFuck program.
 *
 * @param   {string} input A string representing program input.
 * @returns {string}       A string representing program output.
 *
 * @example
 * > let input = '+'.repeat(65) + '.'
 * > msf(input)
 * A
 *
 * @example
 * > input = '+'.repeat(65) + '.' + '+.'.repeat(25)
 * > msf(input)
 * ABCDEFGHIJKLMNOPQRSTUVWXYZ
 */
const msf = function(input) {
    const data = { mem: 0, out: '' };
    [...input].forEach(elt => msfEval(data, elt));
    return data.out;
};
```


## Unit tests (Mocha)

```js
const assert = require('assert');


describe('msfIncr()', function() {
    it('should increment a value less than 256', function() {
        assert.equal(msfIncr(1, 256), 2);
    });
    it('should wrap to 0 if incremented value is 256', function() {
        assert.equal(msfIncr(255, 256), 0);
    });
});


describe('msfEval()', function() {
    const _data = { mem: 0, out: '' };
    let i = 65;
    while (i--) msfEval(_data, '+');
    msfEval(_data, '.');
    it('should increment memory', function () {
        assert.equal(_data.mem, 65);
    });
    it('should add output', function () {
        assert.equal(_data.out, 'A');
    });
});


describe('msf()', function() {
    it('should interpret a single character', function() {
        const _input = '+'.repeat(65) + '.';
        assert.equal(msf(_input), 'A');
    });
    it('should interpret a string', function () {
        const _input = '+'.repeat(65) + '.' + '+.'.repeat(25);
        assert.equal(msf(_input), 'ABCDEFGHIJKLMNOPQRSTUVWXYZ');
    });
});
```
