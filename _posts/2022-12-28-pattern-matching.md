---
title: 'Pattern matching'
date: 2022-12-28
tags: ocaml
src-author: John Whitington
src-title: '<em>OCaml from the Very Beginning</em>, Chapter 3: Case by Case'
src-url: https://johnwhitington.net/ocamlfromtheverybeginning/split06.html
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Question 1

> Rewrite the `not` function from the previous chapter in pattern
> matching style.

```ocaml
let not_orig x =
    if x then false else true

let not x =
    match x with
        true -> false
        | _ -> true

let _ =
    assert (false = not_orig true);
    assert (true = not_orig false);
    assert (false = not true);
    assert (true = not false);
    ;;
```

## Question 2

> Use pattern matching to write a recursive function which, given a
> positive integer `n`, returns the sum of all the integers from 1 to
> `n`.

```ocaml
let rec sum n =
    match n < 1 with
        true -> 0
        | false -> n + sum (n - 1)

let _ =
    assert (sum ~-1 = 0);
    assert (sum 0 = 0);
    assert (sum 1 = 1);
    assert (sum 2 = 3);
    assert (sum 9 = 45);
    ;;
```

## Question 3

> Use pattern matching to write a function which, given two numbers `x`
> and `n`, computes `x^n`.

```ocaml
let rec power x n =
    match n with
        0 -> 1
        | _ -> x * power x (n - 1)

let _ =
    assert (power 0 1 = 0);
    assert (power 1 0 = 1);
    assert (power 1 1 = 1);
    assert (power 2 1 = 2);
    assert (power 1 2 = 1);
    assert (power 2 2 = 4);
    assert (power 2 3 = 8);
    ;;
```

## Question 4

> For each of the previous three questions, comment on whether you think
> it is easier to read the function with or without pattern matching.
> How might you expect this to change if the functions were much larger?

For me there is no difference in these particular cases; but for any
function involving nested if...then clauses, I would find pattern
matching easier to read.

## Question 5

> What does `match 1 + 1 with 2 -> match 2 + 2 with 3 -> 4 | 4 -> 5`
> evaluate to?

I expanded the expression to prevent the warning "8 [partial-match]: this
pattern-matching is not exhaustive."

```ocaml
let _ =
    assert (5 =
        match 1 + 1 with
            2 ->
                (match 2 + 2 with
                    3 -> 4 
                    | 4 -> 5
                    | _ -> 0)
            | _ -> 0
    );
    ;;
```

## Question 6

> There is a special pattern `x..y` to denote continuous ranges of
> characters, for example `'a'..'z'` will match all lowercase letters.
> Write functions `islower` and `isupper`, each of type `char → bool`,
> to decide on the case of a given letter.

```ocaml
let islower c =
    match c with 'a'..'z' -> true
    | _ -> false

let isupper c =
    match c with 'A'..'Z' -> true
    | _ -> false

let _ =
    assert (islower 'a');
    assert (not (islower 'A'));
    assert (isupper 'A');
    assert (not (isupper 'a'));
    ;;
```

## Execute this file

```txt
$ codedown ocaml < 2022-12-28-pattern-matching.md | grep . | ocaml -stdin
```
