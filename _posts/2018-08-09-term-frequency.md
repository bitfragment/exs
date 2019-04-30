---
title: "Term frequency (archaic)"
date: 2018-08-09
tags: python, core
---

Source: [crista/exercises-in-programming-style: Comprehensive collection of programming styles using a simple computational task, term frequency](https://github.com/crista/exercises-in-programming-style)

* TOC
{:toc}

## Notes

Chapter 1 exercise. Task here is to use primary addressable memory.

Lopes:
> In this style, the program reflects the constrained computing environment
> where it executes. The memory limitations force the programmer to come up with
> ways of rotating data through the available memory... Additionally, the
> absence of identifiers results in programs where the natural terminology of
> the problem is absent from the program text, and, instead, is added through
> comments and documentation. This is what programming was all about in the
> early 1950s. This style of programming, however, is not extinct: it is still
> in use today when dealing directly with hardware and when optimizing the use
> of memory.
>
> This style of programming came directly from the first viable model of
> computation, the Turing Machine. A Turing Machine consists of an unbounded
> modifiable state "tape" (the data memory) and a state machine that reads and
> modifies that state.


## Solution

```py
import os, string, sys


# Helper function that creates an intermediate secondary memory.
def touchopen(filename, *args, **kwargs):
    try:
        os.remove(filename)
    except OSError:
        pass
    open(filename, 'a').close()
    return open(filename, *args, **kwargs)


# Primary memory.
data = []


# Load stop words into primary memory.
# Store stop words in an array in `data[0]`.
with open('./stop_words.txt') as f:
    data = [f.read(1024).split(',')]
    assert data[0][0] == 'a'


# Initialize rest of primary memory.
data.append([])     # data[1], for a max 80-character line
data.append(None)   # data[2], for index of first char of a word
data.append(0)      # data[3], for index of characters
data.append(False)  # data[4], for flag indicating a word is found
data.append('')     # data[5], for a word
data.append('')     # data[6], for word, NNNN (4-digit freqency)
data.append(0)      # data[7], for word frequency


# Access secondary memory.
word_freqs = touchopen('/tmp/word_freqs', 'rb+')
assert os.path.isfile('/tmp/word_freqs')


# Open input file and process lines of text.
txt = sys.argv[1]
f = open(txt)

linecount = 0
print(f'Processing {txt}...')

while True:
    # This isn't a fast script, so we need a progress indicator.
    print(f'{linecount} lines     ', end='\r')
    sys.stdout.flush()
    linecount += 1

    # Read a line of input; bail if it's a blank line (indicating EOF).
    # Otherwise, add `\n` if it's missing, and initialize the
    # word-character index and line-character index positions.
    data[1] = [f.readline()]
    if data[1] == ['']:   # end of input file
        break
    if data[1][0][len(data[1][0]) - 1] != '\n':
        data[1][0] = data[1][0] + '\n'
    data[2] = None
    data[3] = 0

    # Examine characters in the line. If we're at the start of a word,
    # set the word index to the line index. If we're inside a word,
    # but the current character isn't an alphanumeric character, then
    # the word has ended.
    for c in data[1][0]:            # data[1][0] is the line
        if data[2] == None:         # we're not yet inside a word...
            if c.isalnum():         # ...so we're at the start of a word
                data[2] = data[3]   # set word index to line index

        else:                       # we're still inside a word
            if not c.isalnum():     # ...but this isn't an alnum char
                data[4] = False     # ...so the word has ended

                # Take this word, as a slice from the word-character
                # index to the line-character index. Continue only if
                # the word is at least two characters in length and is
                # not a stop word.
                data[5] =  data[1][0][data[2]:data[3]].lower()

                if len(data[5]) >= 2 and data[5] not in data[0]:
                    # *What we do if we continue:* the secondary
                    # memory contains accumulated frequency data, so
                    # search it for this word. If it's found,
                    # increment count for this word. Proceed as follows:
                    # 1. Load line from secondary memory.
                    # 2. If it's blank, we're done.
                    # 3. If it has a frequency entry, load it, dividing
                    #    its data between `data[7]` (a numeric value
                    #    representing the word's frequency of
                    #    occurrence) and  `data[6]` (the string
                    #    representing the word itself).
                    # 4. If our word is the word in memory, increment
                    #    the count, set the "word found" flag, and we're
                    #    done searching.
                    while True:
                        data[6] = word_freqs.readline().decode('utf8').strip()
                        if data[6] == '':
                            break
                        data[7] = int(data[6].split(',')[1])
                        data[6] = data[6].split(',')[0].strip()

                        if data[5] == data[6]:
                            data[7] += 1
                            data[4] = True
                            break

                    # If we haven't found our word in memory, write a
                    # new frequency entry at the current position in the
                    # file (its end, since we completed a search of the
                    # entire file). If we *have* found our word in
                    # memory, update its frequency entry by rewriting
                    # it.
                    #
                    # (Here, Lopes's Python 2 code for writing word
                    # frequency entries to `word_freqs` needed updating
                    # to Python 3 syntax for writing a string as bytes.)
                    if not data[4]:
                        word_freqs.seek(0, 1)   # stay at current pos
                        entry = '%20s,%04d\n'.encode('utf8') % \
                            (data[5].encode('utf8'), 1)
                        word_freqs.write(bytes(entry))
                    else:
                        word_freqs.seek(-26, 1) # back up to entry start
                        entry = '%20s,%04d\n'.encode('utf8') % \
                            (data[5].encode('utf8'), data[7])
                        word_freqs.write(bytes(entry))

                    # Return to the beginning of the file.
                    word_freqs.seek(0, 0)

                # Reset the word-character index.
                data[2] = None

        # Increment the line-character index and continue finding
        # words. (Loop block ends here.)
        data[3] += 1

# Processing of input file is complete.
f.close()
word_freqs.flush()

# Flush memory, since we won't need its contents.
del data[:]

# Tabulate 25 most frequently occurring words.
# Set up 25 locations, which we'll use for the 25 most frequently
# occurring words.
data = data + [[]] * (25 - len(data))

# Append locations to hold the current word frequency data.
data.append('')   # data[25] is word,freq from file
data.append(0)    # data[26] is freq

# Load word data from secondary memory, parse it, then compare it
# to the values already tabulated.
while True:
    data[25] = word_freqs.readline().decode('utf-8').strip()
    if data[25] == '':    # EOF, we're done.
        break
    data[26] = int(data[25].split(',')[1])
    data[25] = data[25].split(',')[0].strip()

    for i in range(25):
        if data[i] == [] or data[i][1] < data[26]:
            data.insert(i, [data[25], data[26]])
            del data[26]
            break

# Print tabulated data and close file.
print()   # don't overwrite progress indicator
for tf in data [0:25]:
    if len(tf) == 2:
        print(f'{tf[0]}: {tf[1]}')

word_freqs.close()
```


## Sample output

`hod.txt` is [Heart of Darkness, by Joseph Conrad](https://www.gutenberg.org/files/219/219-h/219-h.htm)

```sh
Processing hod.txt...
3305 lines
out: 153
one: 144
very: 125
kurtz: 122
up: 120
man: 114
see: 92
know: 88
more: 78
time: 78
over: 71
seemed: 69
made: 67
back: 65
came: 63
don: 63
before: 62
little: 62
though: 61
river: 60
looked: 56
upon: 56
well: 52
something: 52
mr: 52
```
