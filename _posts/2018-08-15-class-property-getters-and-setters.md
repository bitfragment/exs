---
title: "Class property getters and setters"
date: 2018-08-15
tags: python
---

Source: Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}

## Notes

Exercise was an introduction to using decorator syntax to
get and set the values of class properties, and to providing
a string representation by implementing `__repr__`.


## My solution

```py
from math import pi

class Circle:
    def __init__(self, radius=1):
        self.radius = radius
        
    def __repr__(self):
        return f'Circle({self.radius})'

    @property
    def radius(self):
        return self.__radius

    @property
    def diameter(self):
        return self.__diameter
    
    @property
    def area(self):
        return self.__area

    @radius.setter
    def radius(self, radius):
        if radius < 0:
            raise ValueError("Radius cannot be negative")
        self.__radius = radius
        self.__diameter = 2 * radius
        self.set_area(radius)

    @diameter.setter
    def diameter(self, diameter):
        self.__diameter = diameter
        self.__radius = diameter / 2
        self.set_area(self.__radius)
        
    def set_area(self, radius):
        self.__area = pi * (radius ** 2)
```


## Provided solutions

Identical to mine.
