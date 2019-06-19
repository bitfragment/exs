---
title: Parser-interpreter
date: 2017-04-18
tags: javascript, core
---

Simple example of a parser-interpreter.

```js
const syntax = {
    '+': function(a, b) {return a + b},
    '-': function(a, b) {return a - b},
    '*': function(a, b) {return a * b},
    '/': function(a, b) {return a / b}
};


const calc = function(exp) {
    let [a, b, op] = exp;
    if (!syntax[op] || [a, b].some(elt => typeof(elt) !== 'number')) {
        return 'Error';
    } else {
        return syntax[op](a, b);
    }
};
```

## Tests

```js
const assert = require('assert');

describe('calc()', function() {
    it("should add", function(){
        assert.equal(calc([1, 2, "+"]), 3);
    });
    it("should subtract", function(){
        assert.equal(calc([1, 2, '-']), -1);
    });
    it("should multiply", function(){
        assert.equal(calc([1, 2, '*']), 2);
    });
    it("should divide", function(){
        assert.equal(calc([1, 2, '/']), 0.5);
    });
    it("should handle errors", function(){
        assert.equal(calc(['1', 2, '+']), 'Error');
        assert.equal(calc([1, 2, '&']), 'Error');
    });
});
```
