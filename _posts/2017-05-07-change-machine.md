---
title: "Change machine"
date: 2017-05-07
tags: javascript
---

sum()
-----

Sum arguments or array.

```js
> sum(1, 2, 3)
6
> sum([1, 2, 3])
6
```

```js
const sum = function(seq) {
    const add = (a, b) => a + b;
    return Array.isArray(seq)
        ? seq.reduce(add)
        : [...arguments].reduce(add);
};
```

mirror()
--------

Mirror an object.

```js
> mirror({'Z': 'A', 'Y': 'B'})
{ A: 'Z', B: 'Y' }
```

```js
const mirror = obj =>
    Object.keys(obj).reduce((acc, curr) => {
        acc[obj[curr]] = curr;
        return acc;
    }, {});
```

Object.values()
---------------

Polyfill for [ES2017 draft Object.values()](https://github.com/tc39/proposal-object-values-entries).

Return an array containing the values in `obj`.

```js
> Object.values({'a': 1, 'b': 2, 'c': 3})
[ 1, 2, 3 ]
```

```js
Object.prototype.values = function(obj) {
    const result = [];
    for (const k of Object.keys(obj)) {
        result.push(obj[k]);
    }
    return result;
};
```

change()
--------

Return change in the fewest possible number of specified return currency units,
returning any non-permitted denominations or objects along with change.

```js
> d  = {'$20': 2000, '$10': 1000, '$5': 500, '$1': 100}
> rd = {'$10': 1000, '$5': 500, '$1': 100}
> change(d, rd, '$20'))
$10 $10
> change(d, rd, '$5'))
$1 $1 $1 $1 $1
```

```js
const change = (denoms, returnDenoms, inserted) => {

    /* We want to be able to look up denomination faces and values
    in both directions. */
    const revLook = mirror(denoms);

    /* When returning change, we need to evaluate denominations in
    descending order. */
    const valsDesc = Object.values(returnDenoms).sort((a, b) => b - a);

    /* To help always make change if possible, by noting if we're not
    yet working with the smallest desired return denomination value. */
    const smallest = valsDesc[valsDesc.length - 1];

    /* Evaluate the input. */
    const input = inserted.split(' ');
    const reject = input.filter(elt => !Object.keys(denoms).includes(elt));
    const accept = input.filter(elt => !reject.includes(elt));
    let total = !accept.length ? 0 : sum(accept.map(elt => denoms[elt]));

    /* Make change. */
    const change = [];
    parseByVal: for (const v of valsDesc) {
        while (total >= v) {

            /* Always make change if possible. */
            if (!change.length && total === v && total > smallest) {
                continue parseByVal;
            }

            change.push(revLook[v]);
            total -= v;
        }
    }
    return change.concat(reject).join(' ');
};
```


Tests
-----

```js
const assert = require('assert');

// Permitted input denominations.
const denoms = {
    '$20': 2000, '$10': 1000,  '$5': 500, '$2': 200, '$1': 100,
    '50¢':   50, '25¢':   25, '10¢': 10
};

// Desired return denominations.
const returnDenoms = {
    '$10': 1000, '$2': 200, '25¢': 25, '10¢': 10
};


describe('change()', function () {

    it('should return unacceptable denominations or objects', function () {
        assert.equal(change(denoms, returnDenoms, '$100'), '$100');
        assert.equal(change(denoms, returnDenoms, 'parking ticket'), 'parking ticket');
    });

    it('should make change in the fewest possible number of desired return currency units', function () {
        assert.equal(change(denoms, returnDenoms, "$1"), "25¢ 25¢ 25¢ 25¢");
        assert.equal(change(denoms, returnDenoms, "$5"), "$2 $2 25¢ 25¢ 25¢ 25¢");
    });

    it('should return unacceptable denominations or objects but still make change', function () {
        assert.equal(change(denoms, returnDenoms, '$1 parking ticket'), '25¢ 25¢ 25¢ 25¢ parking ticket');
    });

    it('should always make change if possible', function () {
        assert.equal(change(denoms, returnDenoms, '50¢'), '25¢ 25¢');
        assert.equal(change(denoms, returnDenoms, "$20"), "$10 $10");
    });

});
```
