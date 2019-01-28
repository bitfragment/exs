---
title: "Poker hand combinations"
date: 2018-12-31
tags: python
---

* TOC
{:toc}


## Description

Generate combinations of 7-card poker hands for testing purposes.

Format is `hand1; hand1`: e.g., `Ad Kd Qd Jd 0d 0d 0d; Ad Kd Qd Jd 0d 4c 4c`.


## My solution

```py
from random import choice

suits = [ 'c', 'd', 'h', 's' ]

values = [str(n) for n in range(2, 10)] + ['0', 'J', 'K', 'Q', 'A']

hands = {
    'royal flush'     : 'Ad Kd Qd Jd 0d',
    'straight flush'  : '8c 7c 6c 5c 4c',
    'four of a kind'  : 'Jh Jd Js Jc 7d',
    'full house'      : '0h 0d 0s 9c 9d',
    'flush'           : '4s Js 8s 2s 9s',
    'straight'        : '9c 8d 7s 6d 5h',
    'three of a kind' : '7c 7d 7s Kc 3d',
    'two pair'        : '4c 4s 3c 3d Qc',
    'pair'            : 'Ah Ad 8c 4s 7h'
}

def randcard():
    return f'{choice(values)}{choice(suits)}'

def randhand():
    hand = ''
    for i in range(6):
        hand += randcard() + ' '
    hand += randcard()
    return hand

total = []

# Pair each ranked hand with itself and every other.
for name1, hand1 in hands.items():
    for name2, hand2 in hands.items():
        rc1, rc2, rc3, rc4 = randcard(), randcard(), randcard(), randcard()
        print(f'{hand1} {rc1} {rc2}; {hand2} {rc4} {rc4}')

# Pair each ranked hand with 3 random hands.
for name1, hand1 in hands.items():
    rs = randhand(), randhand(), randhand()
    rc1, rc2 = randcard(), randcard()
    for hand2 in rs:
        print(f'{hand1} {rc1} {rc2}; {hand2}')

# Generate 10 pairs of random hands.
for i in range(10):
    hand1, hand2 = randhand(), randhand()
    print(f'{hand1}; {hand2}')
```
