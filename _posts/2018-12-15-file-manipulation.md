---
title: "File manipulation"
date: 2018-12-15
tags: python
---

Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}

## Description

Take a file object representing a file containing diary entries in this
format:

```
2018-01-01

Coded.

Did laundry.

2018-01-02

Slept all day.
```

Return entries as follows:

```
>>> diary_file = open('my_diary.txt')
>>> entries_by_date(diary_file)
[("2018-01-01", "Coded.\n\nDid laundry."), ("2018-01-02", "Slept all day.")]
```

Bonus 1: Unescape HTML `&nbsp;`, `&quot;`, and `&amp;`.

Bonus 2: Take a command-line argument representing a diary filename
and create a new file for each diary entry.


## Notes

None.


## My solution

```py
from re import fullmatch, sub


def clean(line):
    '''Unescape HTML `&nbsp;`, `&quot;`, and `&amp;`.'''
    entities = {
        "&nbsp;": ' ',
        "&quot;": '\"',
        "&amp;": '&'
    }
    for k in entities.keys():
        line = sub(k, entities[k], line)
    return line


def get_entry_date(line):
    '''Return the post date or an empty string.'''
    ret = ''
    try:
        ret = fullmatch('[0-9]{4}-[0-9]{2}-[0-9]{2}', line.rstrip()).group()
    except AttributeError:
        pass
    return ret


def entries_by_date_str(lines):
    '''Parse `lines` and return a list of tuples.
    
    Each tuple contains two strings: the entry date and the entry body.
    The entry body contains no whitespace or newlines at its beginning
    or end, but preserves whitespace and newlines otherwise.
    
    If the first line is not a date, returns an empty list.
    '''
    entries = []
    entry_body = []
    try:
        entry_date = get_entry_date(lines[0])
    except IndexError:
        return entries
    newdate = True # if we have just encountered a new date
    for line in lines[2:]:
        next_date = get_entry_date(clean(line))
        if next_date:
            newdate = True
            entry = (entry_date, ''.join(entry_body).rstrip())
            entries.append(entry)
            entry_body = []
            entry_date = next_date
        else:
            if newdate:
                entry_body.append(clean(line).lstrip())
                newdate = False
            else:
                entry_body.append(clean(line))
    entry = (entry_date, ''.join(entry_body).rstrip())
    entries.append(entry)
    return entries


def entries_by_date(file_obj):
    '''Accept a file object. Wrapper for `entries_by_date_str()`.'''
    return entries_by_date_str(list(file_obj))


def main(filename):
    with open(filename) as f:
        entries = entries_by_date(f)
        for entry in entries:
            with open(f'{entry[0]}.txt', 'w') as f:
                f.write(entry[1])
                f.close()



## Interaction

if __name__ == '__main__':
    print("My tests:")
    import pytest
    pytest.main([__file__])

    print("Pymorsel tests:")
    from os import system
    system('python test_reformat_diary.py')



## Tests

def test_clean():
    s = 'I said &quot;rabbit,&nbsp;rabbit&quot; today &amp; burped.'
    assert clean(s) == 'I said "rabbit, rabbit" today & burped.'

def test_get_entry_date_empty():
    assert get_entry_date('foo bar') == ''

def test_get_entry_date_dwim():
    assert get_entry_date('2018-12-09') == '2018-12-09'

def test_get_entry_date_whtspc():
    '''Input includes whitespace.'''
    assert get_entry_date('2018-12-09 ') == '2018-12-09'
    assert get_entry_date('2018-12-09\n') == '2018-12-09'

def test_entries_by_date_str_0():
    txt = []
    assert entries_by_date_str(txt) == []

def test_entries_by_date_str_1():
    txt = ['2018-12-09\n', '\n', 'Foo.\n', '\n', 'Bar.\n']
    assert entries_by_date_str(txt) == [('2018-12-09', 'Foo.\n\nBar.')]

def test_entries_by_date_str_2():
    txt = ['2018-12-09\n', '\n', 'Foo.\n', '\n', 'Bar.\n', '\n',
           '2018-12-10\n', '\n', 'Baz.\n', '\n', 'Qux.\n']

    assert entries_by_date_str(txt) == [('2018-12-09', 'Foo.\n\nBar.'),
                                        ('2018-12-10', 'Baz.\n\nQux.')]
def test_entries_by_date_str_3():
    txt = ['2018-12-09\n', '\n', 'Foo.\n', '\n', 'Bar.\n', '\n',
           '2018-12-10\n', '\n', 'Baz.\n', '\n', 'Qux.\n', '\n',
           '2018-12-11\n', '\n', 'Corge.\n', '\n', 'Grault.\n']

    assert entries_by_date_str(txt) == [('2018-12-09', 'Foo.\n\nBar.'),
                                        ('2018-12-10', 'Baz.\n\nQux.'),
                                        ('2018-12-11', 'Corge.\n\nGrault.')]

def test_entries_by_date():
    with open('my_diary.txt') as f:
        assert entries_by_date(f) == \
            [("2018-01-01", "Coded.\n\nDid laundry."), 
            ("2018-01-02", "Slept all day.")]
    f.close()

def test_main():
    main('my_diary.txt')
    with open('2018-01-01.txt') as f1:
        assert list(f1) == ['Coded.\n', '\n', 'Did laundry.']
        f1.close()
    with open('2018-01-02.txt') as f2:
        assert list(f2) == ['Slept all day.']
        f2.close()
```


## Provided solution

In the provided solution, `entries_by_date()` is a generator function ---
which makes sense, though I'm not sure why it might be better.

The provided solution suggests various ways to strip out HTML entities,
including this one, which appeals to me but feels hard to read. It passes
a lambda function as argument to `re.sub()`.

```py
REPLACEMENTS = {
    " ": " ",
    "&quot;": '"',
    "&": '&',
}

REPLACEMENTS_RE = re.compile('|'.join(REPLACEMENTS.keys()))

def clean_entry(text):
    return REPLACEMENTS_RE.sub(lambda m: REPLACEMENTS[m.group()], text).strip()
```
