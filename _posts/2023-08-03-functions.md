---
title: 'Functions'
date: 2023-08-03
tags: ocaml
src-author: John Whitington
src-title: '<em>OCaml from the Very Beginning</em>, Chapter 6: Functions upon Functions upon Functions'
src-url: https://johnwhitington.net/ocamlfromtheverybeginning/split11.html
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Question 1

> Write a simple recursive function `calm` to replace exclamation marks in
> a **char list** with periods. For example `calm [’H’; ’e’; ’l’; ’p’; ’!’; ’
> ’; ’F’; ’i’; ’r’; ’e’; ’!’]` should evaluate to `[’H’; ’e’; ’l’; ’p’;
> ’.’; ’ ’; ’F’; ’i’; ’r’; ’e’; ’.’]`. Now rewrite your function to use
> `map` instead of recursion. What are the types of your functions?

### My solution

Recursive function:

```ocaml
let rec calm l = match l with
  [] -> []
  | h::t -> match h with
    '!' -> '.' :: calm t
    | _ -> h :: calm t

let _ =
  assert ([] = calm []);
  assert (['x'] = calm ['x']);
  assert (not (['x'] = calm ['!']));
  assert (['.'] = calm ['!']);
  assert (['.'; '.'] = calm ['!'; '!']);
  assert (['.'; 'x'; '.'] = calm ['!'; 'x'; '!']);
  assert (['.'; '.'; 'x'] = calm ['!'; '!'; 'x']);
  assert (['H'; 'e'; 'l'; 'p'; '.'; ' '; 'F'; 'i'; 'r'; 'e'; '.'] = calm ['H'; 'e'; 'l'; 'p'; '!'; ' '; 'F'; 'i'; 'r'; 'e'; '!']);
  ;;
```

Version using `map`:

```ocaml
let rec map f l = match l with
  [] -> []
  | h::t -> f h :: map f t

let calm_map l = match l with
  [] -> []
  | h::t -> map (fun x -> match x with '!' -> '.' | _ -> x) l

let _ =
  assert ([] = calm_map []);
  assert (['x'] = calm_map ['x']);
  assert (not (['x'] = calm_map ['!']));
  assert (['.'] = calm_map ['!']);
  assert (['.'; '.'] = calm_map ['!'; '!']);
  assert (['.'; 'x'; '.'] = calm_map ['!'; 'x'; '!']);
  assert (['.'; '.'; 'x'] = calm_map ['!'; '!'; 'x']);
  assert (['H'; 'e'; 'l'; 'p'; '.'; ' '; 'F'; 'i'; 'r'; 'e'; '.'] = calm_map ['H'; 'e'; 'l'; 'p'; '!'; ' '; 'F'; 'i'; 'r'; 'e'; '!']);
  ;;
```

The types of my functions are:

`calm` : **char list** → **char list**

`calm-map`: **char list** → **char list**

### Solution provided in book

...is more compact and straightforward in both cases.

```ocaml
let rec calm_solution l = match l with
  [] -> []
  | '!'::t -> '.' :: calm_solution t
  | h::t -> h :: calm_solution t

let calm_char_solution x = match x with '!' -> '.' | _ -> x

let calm_map_solution l = map calm_char_solution l
```

## Question 2

> Write a function `clip` which, given an integer, clips it to the range
> 1...10 so that integers bigger than 10 round down to 10, and those
> smaller than 1 round up to 1. Write another function `cliplist` which
> uses this first function together with map to apply this clipping to a
> whole list of integers.

### My solution

```ocaml
let rec map f l =
    match l with [] -> []
    | h::t -> f h :: map f t

let clip x =
  if x < 1 then 1
  else if x > 10 then 10
  else x

let cliplist l = match l with [] -> []
  | _ -> map clip l

let _ =
  assert (1 = clip (-1));
  assert (1 = clip 0);
  assert (1 = clip 1);
  assert (2 = clip 2);
  assert (9 = clip 9);
  assert (10 = clip 10);
  assert (10 = clip 11);
  assert ([] = cliplist []);
  assert ([1] = cliplist [-1]);
  assert ([1; 1] = cliplist [-1; 1]);
  assert ([1; 1] = cliplist [-1; 0]);
  assert ([1; 1] = cliplist [0; 1]);
  assert ([1; 2] = cliplist [0; 2]);
  assert ([1; 2; 1] = cliplist [0; 2; -1]);
  assert ([1; 10; 1] = cliplist [0; 12; -1]);
;;
```

### Solution provided in book

Solution for `clip` is identical to my solution.

Solution for `cliplist` demonstrates that it wasn't necessary for me to
match an empty list, because the `map` function does that itself.

```ocaml
let clip x =
  if x < 1 then 1 else
    if x > 10 then 10 else x

let cliplist l = map clip l
```

## Question 3

> Express your function `cliplist` again, this time using an anonymous
> function instead of `clip`.

### My solution

```ocaml
let cliplist_anon l = map (
  fun x -> if x < 1 then 1 else if x > 10 then 10 else x
  ) l

let _ =
  assert ([] = cliplist_anon []);
  assert ([1] = cliplist_anon [-1]);
  assert ([1; 1] = cliplist_anon [-1; 1]);
  assert ([1; 1] = cliplist_anon [-1; 0]);
  assert ([1; 1] = cliplist_anon [0; 1]);
  assert ([1; 2] = cliplist_anon [0; 2]);
  assert ([1; 2; 1] = cliplist_anon [0; 2; -1]);
  assert ([1; 10; 1] = cliplist_anon [0; 12; -1]);
;;
```

### Solution provided in book

Is identical to my solution. 

## Question 4

> Write a function `apply` which, given another function, a number of
> times to apply it, and an initial argument for the function, will
> return the cumulative effect of repeatedly applying the function. For
> instance, `apply f 6 4` should return `f (f (f (f (f (f 4))))))`. What
> is the type of your function?

```ocaml
let rec apply f n arg =
  match n < 1 with
  | true -> arg
  | _ -> match n with 
    | 1 -> f arg
    | _ -> apply f (n - 1) (f arg)

let f x = x + 1 

let _ =
  assert (0  = apply f (-1)    0);
  assert (0  = apply f   0     0);
  assert (1  = apply f   1     0);
  assert (2  = apply f   2     0);
  assert (3  = apply f   3     0);
  assert (6  = apply f   3     3);
  assert (13 = apply f  10     3);
  assert (49 = apply f  50  (-1));
  ;;
```

The type of `apply` is *(ɑ → β) → ɑ int → β → γ*

Comment on my solution: I wonder what is idiomatic in Ocaml programming
in such case. I don't particularly like mixing `if...then` and `match`
expressions, so after writing a few different versions of this solution,
I settled on `match n < 1 with true` as the expression I liked best to
handle useless arguments for `n`.

### Solution provided in book

...is much more elegant, but overflows the stack with my test cases, 
presuambly because it can't handle a negative argument.

```ocaml
let rec apply f n x =
  if n = 0 then x else f (apply f (n - 1) x)
```

## Question 5

> Modify the insertion sort function from the preceding chapter to take
> a comparison function, in the same way that we modified merge sort in
> this chapter. What is its type?

### My solution

```ocaml
let rec insert cmp x l =
    match l with [] -> [x]
    | h::t ->
        if (cmp x h) then x :: h :: t
        else h :: insert cmp x t
    ;;

let rec sort cmp l =
    match l with [] -> []
    | h::t -> insert cmp h (sort cmp t)
    ;;

let f x y = x <= y

let _ =
    assert ([] = sort f []);
    assert ([0] = sort f [0]);
    assert ([0; 1] = sort f [0; 1]);
    assert ([0; 1] = sort f [1; 0]);
    assert ([0; 1; 2] = sort f [2; 1; 0]);
    assert ([0; 1; 2] = sort f [2; 0; 1]);
    assert ([0; 1; 2] = sort f [1; 0; 2]);
    assert ([0; 1; 2] = sort f [1; 2; 0]);
    assert ([2; 6; 9; 19; 53] = sort f [53; 9; 2; 6; 19]);
    ;;

let f x y = x >= y

let _ = 
  assert ([1; 0] = sort f [0; 1]);
  assert ([2; 1; 0] = sort f [0; 2; 1]);
  assert ([53; 19; 9; 6; 2] = sort f [53; 9; 2; 6; 19]);
  ;;
```

### Solution provided in book

Is identical to mine.

## Question 6

> Write a function **filter** which takes a function of type **α →
> bool** and an **α list** and returns a list of just those elements of
> the argument list for which the given function returns `true`.

### My solution

I wrote two versions, one using `match` syntax exclusively, and one
mixing it with `if...else` syntax. The second seems more readable, but
I don't like mixing two different conditional paradigms.

```ocaml
let rec filter f l =
  match l with [] -> []
  | h::t ->
    match f h with true -> h :: filter f t
    | _ -> filter f t

let rec filter f l =
  match l with [] -> []
  | h::t ->
    if (f h) then h :: filter f t
    else filter f t

let f x = x < 3

let _ =
  assert ([] = filter f []);
  assert ([0] = filter f [0]);
  assert ([0; 1] = filter f [0; 1]);
  assert ([0; 1; 2] = filter f [0; 1; 2]);
  assert ([0; 1; 2] = filter f [0; 1; 2; 3]);
  assert ([0; 1; 2] = filter f [0; 1; 2; 3; 4]);
  assert ([0; 1; 2; 2] = filter f [5; 4; 0; 4; 9; 1; 2; 3; 3; 9; 4; 2]);
  ;;
```

### Solution provided in book

Is identical to my second solution above.

```ocaml
let rec filter f l =
  match l with [] -> []
  | h::t ->
    if f h then h :: filter f t
    else filter f t
```

## Question 7

> Write the function `for_all` which, given a function of type **α →
> bool** and an argument list of type **α list** evaluates to `true` if
> and only if the function returns `true` for every element of the list.
> Give examples of its use.

### My solution

```ocaml
let rec for_all f l =
  match l with [] -> false (* reject meaningless argument *)
  | h::[] -> f h           (* list with exactly one element *)
  | h::t ->
    if f h then for_all f t
    else false

let f x = x < 3

let _ =
  assert (false = for_all f []);
  assert (true = for_all f [(-1)]);
  assert (true = for_all f [0]);
  assert (true = for_all f [0; 1]);
  assert (true = for_all f [0; 1; 2]);
  assert (false = for_all f [0; 1; 2; 3]);
  ;;
```

### Solution provided in book

...is far more elegant than mine, using the logical conjunction `&&`
to limit recursion (by returning as soon as `f h` evaluates to false):

```ocaml
let rec for_all f l =
  match l with
    [] -> true
    | h::t -> f h && for_all f t
```

However, this solution will evaluate an empty list as `true`. That's
bad:

```ocaml
let _ = assert (true = for_all (fun x -> x < 1) []);;
```

That's why I deliberately addressed that case in my own solution.

## Question 8

> Write a function `mapl` which maps a function of type **α → β** over a list
> of type **α list list** to produce a list of type **β list list**.

### My solution

```ocaml
let rec map f l = match l with
  [] -> []
  | h::t -> f h :: map f t

let rec mapl f l = match l with
  [] -> []
  | h::t -> map f h :: mapl f t

let f x = x + 1

let _ =
  assert ([] = mapl f []);
  assert ([[]; []] = mapl f [[]; []]);
  assert ([[1]; [1]] = mapl f [[0]; [0]]);
  assert ([[1; 2]; [3; 4]] = mapl f [[0; 1]; [2; 3]]);
  ;;
```

### Solution provided in book

Is identical to mine.

## Execute this file

```txt
$ codedown ocaml < 2023-08-03-functions.md | grep . | ocaml -stdin
```
