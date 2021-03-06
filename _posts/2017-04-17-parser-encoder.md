---
title: Parser-encoder
date: 2017-04-17
tags: javascript, core
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Argument syntax

- `number * count`. Example: `"5*3"` ⟹ `[5, 5, 5]`
- `begin-end`. Example: `"1-3"` ⟹ `[1, 2, 3]`
- `begin-end/step`. Example: `"1-5/2"` ⟹ `[1, 3, 5]`

`"1, 2-10/2, 5*3"` ⟹ `[ 1, 2, 4, 6, 8, 10, 5, 5, 5 ]`


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


## syntax()

Hash table containing regular expressions and routines for
parsing them.

```js
const syntax = {
    // `number * count`
    '^\-?[0-9]+\\*\-?[0-9]+$': function(s) {
        let [n, rep] = s.split('*').map(Number);
        return Array.apply(null, Array(rep)).map(elt => n);
    },
    // `begin-end`
    '^\-?[0-9]+\-{1}\-?[0-9]+$': function(s) {
        let a = Number(/^\-?[0-9]+/.exec(s)[0]);
        let b = Number(/\-(\-?[0-9]+)$/.exec(s)[1]);
        return Array.from(range(a, b));
    },
    // `begin-end/step`
    '^\-?[0-9]+\-{1}\-?[0-9]+\\/\-?[0-9]+$': function(s) {
        let a = Number(/^\-?[0-9]+/.exec(s)[0]);
        let b = Number(/\-(\-?[0-9]+)\//.exec(s)[1]);
        let step = Number(/\/(\-?[0-9]+)$/.exec(s)[1]);
        return Array.from(range(a, b, step));
    }
};
```


## parse()

```js
const parse = function(s) {
    for (let exp of Object.keys(syntax)) {
        if (new RegExp(exp).test(s)) {
            return syntax[exp](s);
        }
    }
    return Number(s);
};
```


## encode()

```js
const encode = s =>
    [].concat.apply([], s.split(',').map(elt => parse(elt.trim())));
```


## Example usage

```js
> encode('1, 2-10/2, 5*3')
[ 1, 2, 4, 6, 8, 10, 5, 5, 5 ]
```
