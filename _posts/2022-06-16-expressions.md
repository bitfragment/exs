---
title: 'Expressions'
date: 2022-06-16
tags: scheme, javascript
src-author: Abelson, Sussman, Sussman; Henz, Wrigstad
src-title: '<em>Structure and Interpretation of Computer Programs</em>, second ed. and JavaScript ed., Chapter 1: Building Abstractions with Procedures'
src-url: https://sicp.sourceacademy.org/chapters/1.1.6.html
---

## Contents
{:.no_toc}

* TOC
{:toc}


## Note

Using the occasion of the 2022 JavaScript edition of *SICP* to work 
through the original again, alongside this new edition.

Reference solutions provided in the [Comparison Edition].

[Comparison Edition]: https://sicp.sourceacademy.org/index.html


## Setup

```scheme
#lang sicp
(#%require rnrs/base-6) ; to get `assert`
```

```js
const assert = require('assert');
```


## Exercise 1.1 (Scheme)

> Below is a sequence of expressions. What is the result printed by the
> interpreter in response to each expression? Assume that the sequence
> is to be evaluated in the order in which it is presented.

My solution:

```scheme
(assert (= 10 10))
(assert (= 12 (+ 5 3 4)))
(assert (= 8 (- 9 1)))
(assert (= 3 (/ 6 2)))
(assert (= 6 (+ (* 2 4) (- 4 6))))

(define a 3)
(define b (+ a 1))

(assert (= 19 (+ a b (* a b))))
(assert (not (= a b)))

(assert (= 4
  (if (and
    (> b a)
    (< b (* a b)))
      b a)))

(assert (= 16
  (cond ((= a 4) 6)
        ((= b 4) (+ 6 7 a))
        (else 25))))

(assert (= 6
  (+ 2 (if (> b a) b a))))

(assert (= 16
  (* (cond ((> a b) a)
           ((< a b) b)
           (else -1))
    (+ a 1))))
```


## Exercise 1.1 (JavaScript)

> Below is a sequence of statements. What is the result printed by the
> interpreter in response to each statement? Assume that the sequence
> is to be evaluated in the order in which it is presented.

My solution:

```js
assert(10 === 10);
assert(12 === 5 + 3 + 4);
assert(8 === 9 - 1);
assert(3 === 6 / 2);
assert(6 === 2 * 4 + (4 - 6));

const a = 3;
const b = a + 1;

assert(19 === a + b + a * b);
assert(!(a === b));
assert(4 === b > a && b < a * b ? b : a);

assert(16 ===
    (a === 4
        ? 6
        : b === 4
            ? 6 + 7 + a
            : 25)
    );

assert(6 === 2 + (b > a ? b : a));

assert(16 === (a > b ? a : a < b ? b : -1) * (a + 1));
```


## Exercise 1.2 (Scheme)

> Translate the following expression into prefix form.

My solution:

```scheme
(assert (= (/ -37 150)
  (/ (+ 5 4 (- 2 (- 3 (+ 6 (/ 4 5)))))
    (* 3 (- 6 2) (- 2 7)))))
```


## Exercise 1.2 (JavaScript)

> Translate the following expression into JavaScript.

My solution:

```js
assert(
    (-37 / 150) === 
    (5 + 4 + (2 - (3 - (6 + (4/5)))))
    / (3 * (6 - 2) * (2 - 7))
)
```


## Exercise 1.3 (Scheme)

> Define a procedure that takes three numbers as arguments and returns
> the sum of the squares of the two larger numbers.

My solution (without using anything not yet introduced in the book):

```scheme
(define (min a b c)
  (cond ((and (<= a b) (<= a c)) a)
        ((and (<= b a) (<= b c)) b)
        (else c)))

(define (e1.3 a b c)
  (define _min (min a b c))
  (cond ((= a _min) (+ (* b b) (* c c)))
        ((= b _min) (+ (* a a) (* c c)))
        (else (+ (* a a) (* b b)))))

(define (test-min)
  (assert (= -1 (min -1 0 1)))
  (assert (= -1 (min 0 0 -1)))
  (assert (= -1 (min 0 -1 1)))
  (assert (=  1 (min 1 2 3)))
  (assert (=  1 (min 3 2 1)))
  (assert (=  1 (min 3 1 2))))

(define (test-e1.3)
  (assert (=  2 (e1.3 0 1 1)))
  (assert (=  1 (e1.3 0 -1 -1)))
  (assert (= 13 (e1.3 1 2 3)))
  (assert (= 25 (e1.3 3 4 2)))
  (assert (= 41 (e1.3 5 4 3)))
)

(test-min)
(test-e1.3)
```

Reference solution provided in the [Comparison Edition]:

> ```scheme
> (define (f x y z)
>    (let ((smallest (if (> x y) (if (> y z) z y) (if (> x z) z x))))
>       (- (+ (square x) (square y) (square z)) (square smallest))))
> ```


## Exercise 1.3 (JavaScript)

> Declare a function that takes three numbers as arguments and returns
> the sum of the squares of the two larger numbers.

My solution (without using anything not yet introduced in the book):

```js
function min(a, b, c) {
    let _min = a;
    if (b <= a && b <= c) _min = b;
    if (c <= a && c <= b) _min = c;
    return _min;
}

function eOnePointThree(a, b, c) {
    let res;
    let _min = min(a, b, c);
    if (a === _min) res = (b*b) + (c*c);
    if (b === _min) res = (a*a) + (c*c);
    if (c === _min) res = (a*a) + (b*b);
    return res;
}

function testMin() {
    assert(min(-1, 0, 1) === -1);
    assert(min(0, 0, -1) === -1);
    assert(min(0, -1, -1) === -1);
    assert(min(0, 1, 2) === 0);
    assert(min(1, 2, 3) === 1);
    assert(min(3, 2, 1) === 1);
    assert(min(3, 1, 2) === 1);
}

function testEOnePointThree() {
    assert(eOnePointThree(0, 1, 1) === 2);
    assert(eOnePointThree(0, -1, -1) === 1);
    assert(eOnePointThree(1, 2, 3) === 13);
    assert(eOnePointThree(3, 4, 2) === 25);
    assert(eOnePointThree(5, 4, 3) === 41);
}

testMin();
testEOnePointThree();
```

Reference solution provided in the [Comparison Edition]:

> ```js
> function f(x, y, z) {
>     return square(x) + square(y) + square(z) -
>         // subtract the square of the smallest
>         square(x > y ? (y > z ? z : y) : (x > z ? z : x));
> }
> ```


## Exercise 1.4 (Scheme)

> Observe that our model of evaluation allows for combinations whose
> operators are compound expressions. Use this observation to describe
> the behavior of the following procedure:
>
> ```scheme
> (define (a-plus-abs-b a b)
>   ((if (> b 0) + -) a b))
> ```

My solution:

```scheme
(define (a-plus-abs-b a b)
  ((if (> b 0) + -) a b))

; Because the subexpression `(> b 0)` will be evaluated using
; applicative- order evaluation, its resolution can be used to choose
; one of the arithemtic operators `+` or `-`. To illustrate:
(assert (=  3 ((if (> 1 0) + -) 1 2)))
(assert (= -1 ((if (> 0 1) + -) 1 2)))

; Thus, we test `a-plus-abs-b` as follows:
(define (test-a-plus-abs-b)
  (assert (= 1 (a-plus-abs-b 0 1)))
  (assert (= 1 (a-plus-abs-b 1 0)))
)

(test-a-plus-abs-b)
```


## Exercise 1.4 (JavaScript)

> Observe that our model of evaluation allows for applications whose
> function expressions are compound expressions. Use this observation to
> describe the behavior of a_plus_abs_b:
>
> ```js
> function plus(a, b) { return a + b; }
> 
> function minus(a, b) { return a - b; }
> 
> function a_plus_abs_b(a, b) {
>     return (b >= 0 ? plus : minus)(a, b);
> }
> ```

My solution:

```js
function plus(a, b) { return a + b; }

function minus(a, b) { return a - b; }

function aPlusAbsB(a, b) {
    return (b >= 0 ? plus : minus)(a, b);
}

// My explanation is the same as for the Scheme solution.
assert( 3 === (1 > 0 ? plus : minus)(1, 2))
assert(-1 === (0 > 1 ? plus : minus)(1, 2))

function testAPlusAbsB() {
    assert(1 === aPlusAbsB(0, 1));
    assert(1 === aPlusAbsB(1, 0));
    assert(2 === aPlusAbsB(1, -1));
}

testAPlusAbsB();
```


## Exercise 1.5 (Scheme)

> Ben Bitdiddle has invented a test to determine whether the interpreter
> he is faced with is using applicative-order evaluation or normal-order
> evaluation. He defines the following two procedures: 
> 
> ```scheme
> (define (p) (p))
>
> (define (test x y)
>   (if (= x 0)
>       0
>       y))
> ```
> Then he evaluates the expression
>
> ```scheme
> (test 0 (p))
> ```
>
> What behavior will Ben observe with an interpreter that uses
> applicative-order evaluation? What behavior will he observe with an
> interpreter that uses normal-order evaluation? Explain your answer.
> (Assume that the evaluation rule for the special form `if` is the same
> whether the interpreter is using normal or applicative order: The
> predicate expression is evaluated first, and the result determines
> whether to evaluate the consequent or the alternative expression.)

My solution:

```scheme
(define (p) (p))

(define (test x y)
  (if (= x 0)
      0
      y))

; Regarding the expression `(test 0 (p))`: an interpreter using
; normal-order evaluation will never evaluate the subexpression `(p)`,
; because it applies the procedure `test 0` first, obtaining the result
; `0`.
;
; An interpreter using applicative-order evaluation, on the other hand,
; will never terminate, because it evaluates the subexpression `(p)`
; *before* applying the procedure `test`.
; 
; The Scheme interpreter uses applicative-order evaluation, so to avoid
; entering an infinite loop, we have to supply an argument whose
; evaluation will terminate, instead:
(define (test-test)
  (assert (= 0 (test 0 1)))
  (assert (= 0 (test 0 0)))
  (assert (= 0 (test 1 0)))
  (assert (= 1 (test 1 1))))

(test-test)
```


## Exercise 1.5 (JavaScript)

> Ben Bitdiddle has invented a test to determine whether the interpreter
> he is faced with is using applicative-order evaluation or normal-order
> evaluation. He declares the following two functions: 
> 
> ```js
> function p() { return p(); }
>
> function test(x, y) {
>   return x === 0 ? 0 : y;
> }
> ```
> Then he evaluates the expression
>
> ```js
> test(0, p());
> ```
>
> What behavior will Ben observe with an interpreter that uses
> applicative-order evaluation? What behavior will he observe with an
> interpreter that uses normal-order evaluation? Explain your answer.
> (Assume that the evaluation rule for the special form `if` is the same
> whether the interpreter is using normal or applicative order: The
> predicate expression is evaluated first, and the result determines
> whether to evaluate the consequent or the alternative expression.)

My solution:

```js
function p() { return p(); }

function test(x, y) {
  return x === 0 ? 0 : y;
}

// Evidently the Node.js interpreter also performs applicative-order
// evaluation of the statement `test(0, p())`, because this is the
// result:
//
// function p() { return p(); }
//                ^
// RangeError: Maximum call stack size exceeded
// 
// To avoid blowing the stack, we need to supply an argument whose
// evaluation will terminate, instead:
function testTest() {
    assert(0 === test(0, 1));
    assert(0 === test(0, 0));
    assert(0 === test(1, 0));
    assert(1 === test(1, 1));
}

testTest();
```


## Execute this file

Scheme:

```txt
$ codedown scheme < 1-1-elements-of-programming-scm.md | grep . \
> /tmp/tmp.scm && /Applications/Racket\ v8.3/bin/racket /tmp/tmp.scm
```

JavaScript:

```txt
$ codedown js < 1-1-elements-of-programming-js.md | grep . | node
```
