---
title: "Sum timestamps"
date: 2018-12-17
tags: python
---

Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}


## Description

Given a list of timestamps, return their sum. Handle timestamps in
either of these formats or a mixture of both:
    
`5:32`

`1:05:32`


## Notes

None.


## My solution

```py
def sum_timestamps(timestamps):
    '''Given a list of timestamps, return their sum.
    
    Takes a list of strings and returns a string. Handles timestamps
    in either or both of MM:SS and HH:MM:SS formats.'''
    splits = [x.split(':') for x in timestamps]
    
    # Prepend '00' hours to any timestamp that omits hours.    
    splits = [['00'] + x if len(x) < 3 else x for x in splits]

    hrs = [int(x[0]) for x in splits]
    mins = [int(x[1]) for x in splits]
    secs = [int(x[2]) for x in splits]

    # Compute minutes based on seconds.
    carry, secs_sum = divmod(sum(secs), 60)
    mins_sum = sum(mins) + carry

    # Compute hours based on minutes.
    try:
        oflow = sum(hrs)
    except NameError:
        oflow = 0
    if mins_sum > 59:
        carry, mins_sum = divmod(mins_sum, 60)
        oflow += carry

    # Format and return a string.
    ss = str(secs_sum).zfill(2)
    mm = str(mins_sum).zfill(2) if oflow else str(mins_sum)
    hh = str(oflow) if oflow else ''
    return f'{hh}:{mm}:{ss}' if oflow else f'{mm}:{ss}'


## Tests

def test_sum_timestamps_nohrs_nocarry():
    assert sum_timestamps(['03:10', '01:00']) == '4:10'

def test_sum_timestamps_nohrs_carry():
    assert sum_timestamps(['5:32', '4:48']) == '10:20'
    assert sum_timestamps(['2:10', '1:59']) == '4:09'

def test_sum_timestamps_nohrs_overflow():
    assert sum_timestamps(['15:32', '45:48']) == '1:01:20'

def test_sum_timestamps_hrs():
    assert sum_timestamps(['6:15:32', '2:45:48']) == '9:01:20'
    assert sum_timestamps(['6:35:32', '2:45:48', '40:10']) == '10:01:30'


## Interaction

if __name__ == '__main__':
    import pytest
    print('My tests')
    pytest.main([__file__])

    print('Pymorsel tests')
    from os import system
    system('python test_sum_timestamps.py')

```


## Provided solution

The provided solutions all include separate functions for
parsing the timestamps, summing them, and formatting the
output string. This makes good sense.

The final version of the parsing function uses regexps, which
doesn't seem necessary to me.

One version of the summing function uses the `datetime` module.
Given the simplicity of the timestamp format here, that doesn't
seem necessary either. The final version of the summing function
uses `divmod()`, as I did.

```py
import re


TIME_RE = re.compile(r'''
    ^
    (?:
        ( \d + )
        :
    )?
    ( \d + )
    :
    ( \d + )
    $
''', re.VERBOSE)


def parse_time(time_string):
    hours, minutes, seconds = TIME_RE.search(time_string).groups()
    return int(hours or 0)*3600 + int(minutes)*60 + int(seconds)


def format_time(seconds):
    minutes, seconds = divmod(seconds, 60)
    hours, minutes = divmod(minutes, 60)
    if hours:
        return f"{hours}:{minutes:02d}:{seconds:02d}"
    else:
        return f"{minutes:01d}:{seconds:02d}"


def sum_timestamps(timestamps):
    return format_time(sum(parse_time(t) for t in timestamps))
```
