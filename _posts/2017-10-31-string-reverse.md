---
title: "Reverse a string"
date: 2017-10-31
tags: racket
---

Given a string, return it reversed.

```
"foobar" ⟹ "raboof"
```


## My solution

```lisp
#lang racket


(define (reverse s)
  (local [(define lstidx
            (sub1 (string-length s)))]
    (cond [(= 0 lstidx) s]
          [else
           (string-append
            (substring s lstidx)
            (reverse (substring s 0 lstidx)))])))


(provide reverse)
```


## Tests

```lisp
#lang racket

(require rackunit rackunit/text-ui "reverse.rkt")

(define reverse-tests
  (test-suite "reverse tests"
 
    (test-case "Should return the string reversed"
      (check-equal? (reverse "foobar") "raboof"))

    ))

(run-tests reverse-tests)
```
