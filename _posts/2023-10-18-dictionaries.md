---
title: 'Dictionaries'
date: 2023-10-18
tags: ocaml
src-author: John Whitington
src-title: '<em>OCaml from the Very Beginning</em>, Chapter 8: Looking Things Up'
src-url: https://johnwhitington.net/ocamlfromtheverybeginning/split13.html
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Question 1

> Write a function to determine the number of different keys in a
> dictionary.

### My solution

```ocaml
let rec count_keys d =
    match d with [] -> 0
    | (k, v)::t -> 1 + count_keys t

let _ = assert (0 = count_keys []);
        assert (1 = count_keys [('a', 1)]);
        assert (2 = count_keys [('a', 1); ('b', 2)]);
        ;;
```

### Solution provided in book

> Since the keys must be unique, the number of different keys is simply
> the length of the list representing the dictionary --- so we can just
> use the usual `length` function.

Presumably this means the `length` function defined in Chapter 4, on lists:

```ocaml
let rec length l =
    match l with [] -> 0
    | _::t -> 1 + length t
```

## Question 2

> Define a function `replace` which is like `add`, but raises
> `Not_found` if the key is not already there.

### My solution

```ocaml
let rec replace k v d =
  match d with [] -> raise Not_found
  | (k', v')::t ->
    if k = k' then (k, v) :: t
    else (k', v') :: replace k v t

let throws_Not_found f = match f () with 
  | exception Not_found -> true | _ -> false

let _ = 
  assert (throws_Not_found (fun () -> replace 0 1 []));
  assert (throws_Not_found (fun () -> replace 3 4 [(0, 1); (1, 2)]));
  assert ([('a', 1)] = replace 'a' 1 [('a', 0)]);
  assert ([('a', 1); ('b', 2)] = replace 'b' 2 [('a', 1); ('b', 0)]);
  ;;
```

### Solution provided in book

...is identical to my solution.

## Question 3

> Write a function to build a dictionary from two equal length lists,
> one containing keys and another containing values. Raise the exception
> `Invalid_argument` if the lists are not of equal length.

### My solution

Using the `length` function defined previously.

```ocaml
let rec make_dict keys vals = match keys, vals with
    [], _::_ | _::_, [] -> raise (Invalid_argument "lists of unequal length")
    | [], [] -> []
    | kh::kt, vh::vt -> (kh, vh) :: make_dict kt vt

let throws_Invalid_argument f = match f () with 
  | exception (Invalid_argument _) -> true | _ -> false

let _ =
  assert (throws_Invalid_argument (fun () -> make_dict [] [1]));
  assert (throws_Invalid_argument (fun () -> make_dict ['a'] []));
  assert (throws_Invalid_argument (fun () -> make_dict ['a'; 'b'] [1]));
  assert ([] = make_dict [] []);
  assert ([('a', 1)] = make_dict ['a'] [1]);
  assert ([('a', 1); ('b', 2)] = make_dict ['a'; 'b'] [1; 2]);
  assert ([('a', 1); ('b', 2); ('c', 3)] = make_dict ['a'; 'b'; 'c'] [1; 2; 3]);
  ;;
```

### Solution provided in book

...is identical to my own, with two usage differences: less specific
pattern matching, and the argument passed to the exception handler.

```ocaml
let rec mkdict keys values = match keys, values with
  [], [] -> []
  | [], _ -> raise (Invalid_argument "mkdict")
  | _, [] -> raise (Invalid_argument "mkdict")
  | k::ks, v::vs -> (k, v) :: mkdict ks vs
```

## Question 4

> Now write the inverse function: given a dictionary, return the pair of
> two lists – the first containing all the keys, and the second
> containing all the values.

### My solution

I did not like the near-redundancy here but couldn't devise any other
logic for the solution.

```ocaml
let rec getkeys d = match d with
  [] -> []
  | (k, _)::t -> k :: getkeys t

let rec getvals d = match d with
  [] -> []
  | (_, v)::t -> v :: getvals t

let rec getdict d = match d with
  [] -> [], []
  | _ -> getkeys d, getvals d

let _ = assert (([], []) = getdict []);
        assert ((['a'], [1]) = getdict [('a', 1)]);
        assert ((['a'; 'b'], [1; 2]) = getdict [('a', 1); ('b', 2)]);
        assert ((['a'; 'b'; 'c'], [1; 2; 3]) = getdict [('a', 1); ('b', 2); ('c', 3)]);
        ;;
```

### Solutions provided in book

This first solution uses pattern matching to obtain both lists at the
same time. Given a non-empty dictionary, it recursively invokes `mklist`
on the tail of that list, meanwhile building a keys list and a values
list from each head element of the dictionary.

While I can explain this, I do not grasp it intuitively.

```ocaml
let rec mklists d = match d with
  [] -> [], []
  | (k, v)::t ->
    match mklists t with (ks, vs) -> k :: ks, v :: vs
```

Second solution:

> Since the inner pattern match has only one form, and is complete, we
> can use **let** instead:

```ocaml
let rec mklists d = match d with
  [] -> [], []
  | (k, v)::t ->
    let (ks, vs) = mklists t in (k :: ks, v :: vs)
```

This change does make it slightly clearer but I still do not grasp it
intuitively.

## Question 5

> Define a function to turn any list of pairs into a dictionary. If
> duplicate keys are found, the value associated with the first
> occurrence of the key should be kept.

### My solution

I first wrote `find` and `drop` functions.

```ocaml
let rec find k d = match d with
  [] -> false
  | (k', v')::t -> if k = k' then true else find k t

let _ = assert (false = find 'a' []);
        assert (false = find 'b' [('a', 1)]);
        assert (true = find 'a' [('a', 1)]);
        assert (true = find 'b' [('a', 1); ('b', 2)]);
        ;;

let rec drop k d = match d with
  [] -> []
  | (k', v')::t -> if k = k' then drop k t else (k', v') :: drop k t

let _ = assert ([] = drop 'a' []);
        assert ([('b', 2)] = drop 'a' [('b', 2)]);
        assert ([('a', 1)] = drop 'b' [('a', 1); ('b', 2)]);
        assert ([('b', 2)] = drop 'a' [('a', 1); ('b', 2)]);
        ;;
```

My first solution was explicit but verbose.

```ocaml
let rec mkdict l = match l with
  [] -> []
  | (k, v)::t ->
    if find k t then (k, v) :: mkdict (drop k t)
    else (k, v) :: mkdict t
```

My second was concise: omit `find`, just use `drop` immediately.

```ocaml
let rec mkdict l = match l with
  [] -> []
  | (k, v)::t -> (k, v) :: mkdict (drop k t)

let _ = assert ([] = mkdict []);
        assert ([('a', 1)] = mkdict [('a', 1)]);
        assert ([('a', 1); ('b', 2)] = mkdict [('a', 1); ('b', 2)]);
        assert ([('a', 1)] = mkdict [('a', 1); ('a', 2)]);
        assert ([('a', 1)] = mkdict [('a', 1); ('a', 2); ('a', 3)]);
        let l = [('a', 1); ('a', 2); ('a', 3); ('b', 4); ('a', 5)] in
          assert ([('a', 1); ('b', 4)] = mkdict l);
        let l = [('a', 1); ('b', 2); ('a', 3); ('b', 4); ('a', 5)] in
          assert ([('a', 1); ('b', 2)] = mkdict l);
        ;;
```

### Solution provided in book

Uses the `member` function defined in Chapter 4 "Making Lists." The
logic is similar to that of my first solution, but it's even more
verbose. On the other hand, in this solution only `keys_seen` is
examined in its entirety within each iteration, whereas in mine, `drop`
examines the entire tail within each iteration. Is that a meaningful
difference?

```ocaml
let rec member e l =  match l with
  [] -> false
  | h::t ->
    match (h, e) with (a, b) when a = b -> true
    | _ -> member e t

let rec dict_of_pairs_inner keys_seen l = match l with
  [] -> []
  | (k, v)::t ->
    if member k keys_seen
      then dict_of_pairs_inner keys_seen t
      else (k, v) :: dict_of_pairs_inner (k :: keys_seen) t

let dict_of_pairs l = dict_of_pairs_inner [] l
```

## Question 6

> Write the function `union a b` which forms the union of two
> dictionaries. The union of two dictionaries is the dictionary
> containing all the entries in one or other or both. In the case that a
> key is contained in both dictionaries, the value in the first should
> be preferred.

### My solution

Using the `drop` function defined above.

```ocaml
let rec union a b = match a, b with
  [], [] -> []
  | (ka, va)::t, [] -> (ka, va) :: union t []
  | [], (kb, vb)::t -> (kb, vb) :: union [] t
  | (ka, va)::ta, (kb, vb)::tb ->
    (* if the keys are identical, or if the key in `b` is already in
       the tail of `a`, drop that key-value pair from the tail of `b`
       before proceeding any further. *)
    if ka = kb || find kb ta
    then (ka, va) :: union ta (drop kb tb)
    else (ka, va) :: (kb, vb) :: union ta tb

let _ = assert ([] = union [] []);
        assert ([('a', 1)] = union [('a', 1)] []);
        assert ([('a', 1)] = union [] [('a', 1)]);
        assert ([('a', 1); ('b', 2)] = union [('a', 1)] [('b', 2)]);
        assert ([('a', 1); ('b', 2)] = union [('a', 1); ('b', 2)] []);
        assert ([('a', 1); ('b', 2)] = union [] [('a', 1); ('b', 2)]);
        assert ([('a', 1); ('b', 2)] = union [('a', 1)] [('a', 1); ('b', 2)]);
        assert ([('a', 1); ('b', 2)] = union [('a', 1)] [('a', 2); ('b', 2)]);
        assert ([('a', 1); ('b', 2)] = union [('a', 1); ('b', 2)] [('a', 2); ('b', 3)]);
        assert ([('a', 2); ('b', 3)] = union [('a', 2); ('b', 3)] [('a', 1); ('b', 2)]);

        (* The preceding tests assume `a` and `b` are both valid dictionaries,
           meaning that no key is duplicated within either dictionary. Now let's
           assume that `b` is an invalid dictionary. That way, we can use `union`
           to unify `a` with a concatenation of multiple, non-unified dictionaries. *)
        let a = [('a', 1); ('b', 2); ('c', 3)] and b = [('c', 0); ('c', 1); ('c', 2)] in
          assert ([('a', 1); ('b', 2); ('c', 3)] = union a b);
        let a = [('a', 1); ('b', 2); ('c', 3)]
          and b = [('a', 0); ('a', 2); ('a', 3); ('b', 0); ('b', 1); ('b', 3); ('c', 0); ('c', 1); ('c', 2)]
          in assert ([('a', 1); ('b', 2); ('c', 3)] = union a b);
        ;;
```

### Solution provided in book

Uses the `add` function defined previously, which replaces a key if it
already exists. Compared to mine this is a far more elegant solution.

> We pattern match on the first list --- if it is empty, the result is
> simply `b`. Otherwise, we add the first element of the first list to
> the union of the rest of its elements and the second list.

```ocaml
let rec add k v d = match d with
  [] -> [(k, v)]
  | (k', v')::t ->
    if k = k' then (k, v) :: t
    else (k', v') :: add k v t

let rec union a b = match a with
  [] -> b
  | (k, v)::t -> add k v (union t b)
```

```txt
$ codedown ocaml < 2023-10-18-dictionaries.md | grep . | ocaml -stdin
```
