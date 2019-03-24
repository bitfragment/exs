---
title: "Context manager"
date: 2019-03-24
tags: python
---

## Source

Dan Bader, 
[Python Tricks: The Book](https://realpython.com/products/python-tricks-book/), 
2.3 Context Managers and the `with` statement


## Description

Implement a context manager that measures the execution time
of a code block using the `time.time()` function. Write two versions:
    
1. a class-based version
2. a generator-based version used with a `contextmanager` decorator 



## My solution

```py
from contextlib import contextmanager
from time import time


class BlockTimer:
    """Class-based version.
    
    Define `__enter__()` and `__exit__()`, which are invoked
    when entering and exiting the context established by the `with`
    keyword.
    
    Also define `__repr__()` to return the string representation used
    by `print()`.
    """
    def __init__(self):
        self.begin = 0
        self.end = 0
        
    def __enter__(self):
        self.begin = time()
        
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.begin:
            self.end = time()
            print(self)

    def __repr__(self):
        return f'Code block executed in {self.end - self.begin :.02f}s'


@contextmanager
def block_timer():
    """Generator-based version.
    
    `yield` to return to context.
    """
    try:
        begin = time()
        yield
    finally:
        print(f'Code block executed in {time() - begin :.02f}s') 
    


if __name__ == '__main__':
    from time import sleep

    with BlockTimer() as bt:
        sleep(3)

    with block_timer() as bt:
        sleep(3)
```
