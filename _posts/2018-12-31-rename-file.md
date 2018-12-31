---
title: "Rename a file"
date: 2018-12-31
tags: py
---

* TOC
{:toc}


## Description

Rename a text file using a string it includes.


## My solution

```py
import os
from re import compile

# Text in document representing the desired name.
name = "<title>(.*)</title>"
name_re  = compile(name)

dirname = '.'
ext = '.html'

for f in os.listdir(dirname):
    p = os.path.join(dirname, f)
    try:
        p_ext = os.path.splitext(p)[-1]
        if p_ext == ext: 
            with open(p, 'r', encoding='utf-8', errors='ignore') as fp:
                contents = fp.read()
            try:
                title = name_re.findall(contents)[0]
            except Exception:
                title = ''
            if title:
                title = title.lower().replace(' ', '-')
                new = os.path.join(dirname, title + p_ext)
                print(f'Renaming {p} -> {new}')
                os.rename(p, new)
    except IsADirectoryError:
        pass
```
