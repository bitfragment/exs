---
title: "Inheritance and polymorphism"
date: 2019-03-04
tags: python
---

Source: Programmr, [Report Info]

[Report Info]: http://www.programmr.com/report-info

* TOC
{:toc}


## Description

"Given a `Person` class, where each person has a name and a city, write a
class `Programmer` which is a subclass of `Person`. A programmer should have a
name, city, and a programming language. Write a `report()` method for
programmer which prints all their data, each on a single line. Use the
overriding principle."


## My solution

```py
class Person:
    def __init__(self, name, city):
        self.name = name
        self.city = city

    def report(self):
        return f'{self.name}, {self.city}'


class Programmer(Person):
    def __init__(self, name, city, proglang):
        super().__init__(name, city)
        self.proglang = proglang

    def report(self):
        return f'{super().report()} ({self.proglang})' 
```
