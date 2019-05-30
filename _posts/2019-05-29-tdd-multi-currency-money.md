---
title: "TDD: multi-currency money"
date: 2019-05-29
tags: python
src-title: "Kent Beck, Test-Driven Development by Example"
src-url: https://www.pearson.com/us/higher-education/program/Beck-Test-Driven-Development-By-Example/PGM206172.html
---

Chapter 1: "Multi-Currency Money"


## Iterations

First test. There is no other code! Fails with "NameError: name 'Dollar' is
not defined".

```py
def test_multiplication():
    five = Dollar(5)
    five.times(2)
    assert five.amount == 10
```

First iteration: add class. Fails with "TypeError: Dollar() takes no
arguments".

```py
class Dollar:
    pass
```

Second iteration: add stub constructor. Fails with "TypeError:
__init__() takes 1 positional argument but 2 were given".

```py
class Dollar:
    def __init__(self):
        pass
```

Third iteration: add constructor parameter. Fails with
"AttributeError: 'Dollar' object has no attribute 'times'".

```py
class Dollar:
    def __init__(self, amount: float):
        pass
```

Fourth iteration: add stub method. Fails with "AttributeError:
'Dollar' object has no attribute 'amount'".

```py
class Dollar:
    def __init__(self, amount: float):
        pass
    
    def times(self, multiplier):
        pass
```

Fifth iteration: add class attribute `amount`. Fails with
AssertionError: assert 5 == 10.

```py
class Dollar:
    def __init__(self, amount: float):
        self.amount = amount
    
    def times(self, multiplier):
        pass
```

Sixth iteration: hard-code a value that will pass the test.
Passes.

```py
class Dollar:
    def __init__(self, amount: float):
        self.amount = 10
    
    def times(self, multiplier):
        pass
```

Seventh iteration: refactor to remove the duplication of the value 10
(once in the test, once in the code). Passes.

```py
class Dollar:
    def __init__(self, amount: float):
        self.amount = 5 * 2
    
    def times(self, multiplier):
        pass
```

Eight iteration: refactor to move the computation from the class
attribute to the method. Passes.

```py
class Dollar:
    def __init__(self, amount: float):
        self.amount = 0
    
    def times(self, multiplier):
        self.amount = 5 * 2
```

Ninth iteration: refactor to get the value 5 from the constructor
argument. Passes.

```py
class Dollar:
    def __init__(self, amount: float):
        self.amount = amount
    
    def times(self, multiplier):
        self.amount *= 2
```

Tenth iteration: refactor to get the value 2 from the method
argument. Passes.

```py
class Dollar:
    def __init__(self, amount: float):
        self.amount = amount
    
    def times(self, multiplier):
        self.amount *= multiplier
```
