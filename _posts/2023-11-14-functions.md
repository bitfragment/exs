---
title: 'Functions'
date: 2023-11-14
tags: ocaml
src-author: John Whitington
src-title: '<em>OCaml from the Very Beginning</em>, Chapter 9: More with Functions'
src-url: https://johnwhitington.net/ocamlfromtheverybeginning/split14.html
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Question 1

> Rewrite the summary paragraph at the end of this chapter for the three
> argument function `g a b c`.

### My solution

The function `g a b c` has type *α → β → γ → δ* which can also be
written *α → (β → (γ → δ))*. Thus, it takes an argument of type *α* and
returns a function of type *β → (γ → δ)* which, when you give it an
argument of type *β* returns a function of type *γ → δ* which, when you
give it an argument of type *γ* returns something of type *δ*. And so,
we can apply just one argument to the function `g` (which is called
partial application) or apply both at once. When we write **`let`** `g a
b c = ...` this is just shorthand for **`let`** `g = ` **`fun`** `a ->`
**`fun`** `b ->` **`fun`** `c -> ...`

### Solution provided in book

...is identical to my solution.

## Question 2

> Recall the function `member x l` which determines if an element `x` is
> contained in a list `l`. What is its type? What is the type of `member x`? 
> Use partial application to write a function `member_all x ls` which
> determines if an element is a member of all the lists in the list of
> lists `ls`.

### My solution

The function `member x l` has type *α → β* **list** *→ γ **bool***. The
type of `member x` is *α **list** → β* **bool**.

I first wrote an `all` function, which returns true if all of the items
in a provided list are evaluated true by a provided function.

```ocaml
let rec all f l = match l with
    [h] -> f h
    | [] -> false
    | h::t -> f h && all f t
```

Then I wrote a simplified `member` function (the example referred to by
the question was intended to verify if a key exists in a dictionary)
that operates on a list.

```ocaml
let rec member x l = match l with
  [] -> false
  | h::t -> if h = x then true else member x t
```

Finally, I defined `member_all` by partially applying `member` and
passing the result of that partial application, which is a function that
takes a list, as the argument to `all` that represents the provided
function.

```ocaml
let member_all x ls = all (member x) ls

let _ = assert (false = member_all 0 [[]]);

        assert (false = member_all 0 [[1]]);
        assert (true  = member_all 0 [[0]]);

        assert (false = member_all 0 [[1]; [2]]);
        assert (false = member_all 0 [[1; 2]; [2; 3]]);

        assert (true = member_all 0 [[0]; [0]]);
        assert (true = member_all 0 [[0; 1]; [0; 2]]);
        ;;
```

I have the sense there's a more elegant solution to this that the book
has in mind. We'll see in a moment.

### Solution provided in book

It turns out I wasn't missing something. This solution is quite different,
using a `map` function to build a list of boolean values, then checking
to see if that list contains `false`.

Version 1:

```ocaml
let rec map f l =
    match l with [] -> []
    | h::t -> f h :: map f t

let member_all x ls =
  let bools = map (member x) ls in not (member false bools)
```

Version 2:

```ocaml
let member_all x ls =
  not (member false (map (member x) ls))
```

The book asks, "Which do you think is clearer?" (153). To me neither is
as clear as my own solution.

## Question 3

> Why can we not write a function to halve all the elements of a list
> like this: `map (( / ) 2) [10; 20; 30]`? Write a suitable division
> function which can be partially applied in the manner we require.

### My solution

Addition and multiplication are commutative:
valid:

```ocaml
let _ =
  assert ([2; 3; 4] = map (( + ) 1) [1; 2; 3]);
  assert ([2; 4; 6] = map (( * ) 2) [1; 2; 3]);
  ;;
```

However, division and subtraction are not commutative. In `map (( / ) 2)
[10; 20; 30]`, the partial application of the anonymous function `( / )`
using the value `2` establishes that `2` is the dividend, rather than
the divisor. Therefore, it returns a function that expects an integer
value and treats that integer value as a divisor: that is, it divides
`2` by that integer value. But that's not what we want: we want `2` to
be treated as the divisor instead.

Here's a division function that can be partially applied in the way we
expect, by specifying that its first argument is the divisor. When
partially applied with an integer value representing a divisor, `mydiv`
returns a function that expects an integer representing a dividend.

```ocaml
let mydiv divisor dividend = dividend / divisor

let _ = assert ([1; 2; 3] = map (mydiv 2) [2; 4; 6]);
        assert ([1; 2; 3] = map (mydiv 3) [3; 6; 9]);
        ;;
```

### Solution provided in book

...is identical to mine.

## Question 4

> Write a function `mapll` which maps a function over lists of lists of
> lists. You must not use the **let rec** construct. Is it possible to write
> a function which works like `map`, `mapl`, or `mapll` depending upon
> the list given to it?

### My solution

```ocaml
let mapll f = map (map (map f))

let _ = assert (
  [[[1; 2]; [3; 4]]; [[5; 6]; [7; 8]]] =
  mapll (fun x -> x + 1) [[[0; 1]; [2; 3]]; [[4; 5]; [6; 7]]]
  )
  ;;
```

With these constraints (using partial application, and avoiding the
**let rec** construct) it is not possible to write a function that works
like `map`, `mapl`, or `mapll` depending upon the list given to it.

### Solution provided in book

...is identical to my solution.

## Question 5

> Write a function `truncate` which takes an integer and a list of lists,
> and returns a list of lists, each of which has been truncated to the
> given length. If a list is shorter than the given length, it is
> unchanged. Make use of partial application.

### My solution

Using the `take` function defined in Chapter 4.

```ocaml
let rec take n l =
  if n = 0 then []
  else match l with
    [] -> [] 
    | h::t -> h :: take (n - 1) t

let truncate x ll = map (take x) ll

let _ = assert ([] = take 0 []);
        assert ([] = take 1 []);
        assert ([] = take 0 [1; 2; 3]);
        assert ([1] = take 1 [1; 2; 3]);
        assert ([1; 2] = take 2 [1; 2; 3]);
        assert ([1; 2; 3] = take 3 [1; 2; 3]);
        assert ([1; 2; 3] = take 4 [1; 2; 3]);
        ;;

let _ = assert ([[]; []]   = truncate 0 [[]; []]);
        assert ([[]; []]   = truncate 0 [[1; 2; 3]; [2; 3; 4]]);
        assert ([[]; []]   = truncate 0 [[]; [2; 3; 4]]);
        assert ([[]; []]   = truncate 0 [[1]; [2; 3; 4]]);
        assert ([[1]; [2]] = truncate 1 [[1; 2; 3]; [2; 3; 4]]);
        assert ([[1; 2; 3]; [2; 3; 4]] = truncate 3 [[1; 2; 3]; [2; 3; 4]]);
        (* Lists of different lengths. *)
        assert ([[1]; [2]] = truncate 1 [[1; 2; 3]; [2; 3; 4; 5; 6]]);
        assert ([[1; 2]; [2; 3]] = truncate 2 [[1; 2; 3]; [2; 3; 4; 5; 6]]);
        assert ([[1; 2; 3]; [2; 3; 4]] = truncate 3 [[1; 2; 3]; [2; 3; 4; 5; 6]]);
        assert ([[1; 2; 3]; [2; 3; 4; 5]] = truncate 4 [[1; 2; 3]; [2; 3; 4; 5; 6]]);
        (* Truncate to a length longer than either of the lists provided.
           This should not fail. *)
        assert ([[]; []]   = truncate 3 [[]; []]);
        assert ([[1; 2; 3]; [2; 3; 4]] = truncate 4 [[1; 2; 3]; [2; 3; 4]]);
        assert ([[1; 2; 3]; [2; 3; 4]] = truncate 5 [[1; 2; 3]; [2; 3; 4]]);
        assert ([[1; 2; 3]; [2; 3; 4; 5; 6]] = truncate 6 [[1; 2; 3]; [2; 3; 4; 5; 6]]);
        ;;
```

### Solution provided in book

Uses the `length` and `take` functions defined previously, whereas my
solution uses only the `take` function. I don't see the need for this
solution. The book remarks on "being careful to deal with the case where
there is not enough [in the lists] to take" (153), but my simpler
solution handles such cases. The book provides a second solution using
exception handling to deal with such cases, but again, that is not
necessary: `take` already handles such cases.

```ocaml
let rec length l =
    match l with
        [] -> 0
    | _::t -> 1 + length t

let truncate_l n l = if length l >= n then take n l else l

let truncate n ll = map (truncate_l n) ll
```

## Question 6

> Write a function which takes a list of lists of integers and returns
> the list composed of all the first elements of the lists. If a list is
> empty, a given number should be used in place of its first element.

### My solution

```ocaml
let first n l = match l with [] -> n | h::_ -> h

let firsts n ll = map (first n) ll

let _ = assert ( 0 = first 0 []     );
        assert ( 1 = first 0 [1]    );
        assert ( 1 = first 0 [1; 2] );
        ;;

let _ = assert ( [0]       = firsts 0 [[]]                          );
        assert ( [0; 0]    = firsts 0 [[]; []]                      );
        assert ( [0; 1]    = firsts 0 [[]; [1]]                     );
        assert ( [1; 0]    = firsts 0 [[1]; []]                     );
        assert ( [1; 1]    = firsts 0 [[1]; [1]]                    );
        assert ( [1; 0; 2] = firsts 0 [[1; 2]; []; [2; 3]]          );
        assert ( [1; 0; 2] = firsts 0 [[1; 2; 3]; []; [2; 3; 4; 5]] );
        ;;
```

### Solution provided in book

...is identical to my solution.

## Execute this file

```txt
$ codedown ocaml < 2023-11-14-functions.md | grep . | ocaml -stdin
```
