---
title: Term frequency (monolithic)
date: 2021-12-10
tags: python, core
src-author: Cristina Videira Lopes
src-title: <em>Exercises in Programming Style</em>
src-url: https://github.com/crista/exercises-in-programming-style
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Note

> This part of the book presents four basic styles: *Monolithic, 
> Cookbook, Pipeline,* and *Code Golf.* These are basic in the sense that
> their presence is pervasive in programming. (Lopes 25)

This is the Monolithic style.

## Description

Constraints: "No named abstractions. No, or little, use of libraries."

> From a design perspective, the main concern is to obtain the desired
> output without having to think much about subdividing the problem or how
> to take advantage of code that already exists.
> 
> In fact, it is not unusual to see long blocks of program text in modern 
> programs. In some cases, those long program texts are warranted, for
> performance reasons or because the task being coded can't be easily
> subdivided.
> 
> Monolithic programs are hard to follow, in the same way that a manual
> without chapter or sections would be hard to follow. (Lopes 29-30)

## Solution

Solution given in the book. I modified variable names, enabled passing a
stop word file as script argument, added a progress indicator, and added
comments.

```py
import string
import sys

stopfile = sys.argv[1]
txtfile = sys.argv[2]
word_freqs = []

with open(stopfile) as sw:
    stop_words = sw.read().split(',')
stop_words.extend(list(string.ascii_lowercase))

# This isn't fast, so we need a progress indicator.
linecount = 0
print(f'Processing {txtfile}...')

# Process the file one line at a time.
for line in open(txtfile):

    linecount += 1
    print(f'{linecount} lines     ', end='\r')
    sys.stdout.flush()

    # This will store the index position of the first character in a word.
    start_idx = None

    # We'll eventually increment this way down below where we can't
    # see it from here.
    char_idx = 0

    # Process the line character by character.
    for c in line:

        # If we're clean...
        if not start_idx:

            # Determine if we're at the beginning of a word.
            # If we are, set `start_char`;
            if c.isalnum():
                start_idx = char_idx
        else:

            # Determine if we're at the end of a word.
            if not c.isalnum():
                found = False

                # Get the word and reformat it.
                word = line[start_idx:char_idx].lower()

                # If this word isn't a stop word, we will tabulate
                # it. Use `wf_idx` to store the index of this word
                # in `word_freqs`.
                if word not in stop_words:
                    wf_idx = 0

                    # Iterate over `wf_idx` looking for this word. If it 
                    # is already in `word_freqs`, increment its count, set
                    # `found` to `True`, and break. Otherwise increment
                    # `wf_idx` and continue.
                    for record in word_freqs:
                        if word == record[0]:
                            record[1] += 1
                            found = True
                            break
                        wf_idx += 1

                    # If this word is not already in `word_freqs`,
                    # append it.
                    if not found:
                        word_freqs.append([word, 1])

                    # If `word_freqs` already has 2 or more entries,
                    # keep it ordered.
                    elif len(word_freqs) > 1:
                        for n in reversed(range(wf_idx)):
                            if word_freqs[wf_idx][1] > word_freqs[n][1]:
                                word_freqs[n], word_freqs[wf_idx] = \
                                    word_freqs[wf_idx], word_freqs[n]
                                wf_idx = n
                
                # We're done with this word. Reset the value of the
                # variable we've been using to preserve the index
                # of its first character.
                start_idx = None
        
        # Prepare to proceed to the next character in the line.
        char_idx += 1

# Display the 25 records with the greatest frequency values.
print()
for tf in word_freqs[0:25]:
    print(f'{tf[0]}: {tf[1]}')
```

## Execute this file

```bash
$ curl https://raw.githubusercontent.com/crista/exercises-in-programming-style/master/stop_words.txt > stop_words.txt
$ curl https://raw.githubusercontent.com/crista/exercises-in-programming-style/master/pride-and-prejudice.txt > pride-and-prejudice.txt
$ python3 03-02.py stop_words.txt pride-and-prejudice.txt
Processing pride-and-prejudice.txt...
13426 lines
mr: 691
very: 473
elizabeth: 446
darcy: 385
such: 368
much: 310
more: 307
mrs: 292
bennet: 292
bingley: 274
one: 265
jane: 259
miss: 255
know: 226
nd: 222
now: 218
well: 212
though: 207
never: 206
before: 205
herself: 204
soon: 199
sister: 199
think: 196
time: 194
```

## Exercise 3.7.2

> *Readlines.* The example program reads one line at a time from the file.
> Modify it so that it first loads the entire file into memory (with
> `readlines()`), and then iterates over the lines in memory.

### My solution

```bash
$ diff 03-02.py 03-07-02.py
```

```diff
17c17
< for line in open(txtfile):
---
> for line in open(txtfile).readlines():
```

## Exercise 3.7.3

> *Two loops.* Within the monolithic style, modify the program so that the 
> reordering is done in a separate loop at the end, before the word
> frequencies are printed out on the screen.

### My solution

```bash
$ diff 03-02.py 03-07-03.py
```

```diff
71,81d70
<                     # If `word_freqs` already has 2 or more entries,
<                     # keep it ordered.
<                     elif len(word_freqs) > 1:
<
<                         # Move this word to the right place in order.
<                         for n in reversed(range(wf_idx)):
<                             if word_freqs[wf_idx][1] > word_freqs[n][1]:
<                                 word_freqs[n], word_freqs[wf_idx] = \
<                                     word_freqs[wf_idx], word_freqs[n]
<                                 wf_idx = n
<
89a79,94
> # Sort records using bubble sort.
> n = len(word_freqs)
>
> # Progress indicator.
> recordcount = 0
> print(f'\nSorting records...')
>
> for i in range(n - 1):
>     recordcount += 1
>     print(f'{recordcount} records     ', end='\r')
>     sys.stdout.flush()
>     for j in range(0, n - i - 1):
>         if word_freqs[j][1] < word_freqs[j + 1][1]:
>             word_freqs[j], word_freqs[j + 1] = \
>                 word_freqs[j + 1], word_freqs[j]
```

```bash
$ python3 03-07-03.py stop_words.txt pride-and-prejudice.txt
Processing pride-and-prejudice.txt...
13426 lines
Sorting records...
8211 records
mr: 691
very: 473
elizabeth: 446
darcy: 385
such: 368
much: 310
more: 307
bennet: 292
mrs: 292
bingley: 274
one: 265
jane: 259
miss: 255
know: 226
nd: 222
now: 218
well: 212
though: 207
never: 206
before: 205
herself: 204
soon: 199
sister: 199
think: 196
time: 194
````