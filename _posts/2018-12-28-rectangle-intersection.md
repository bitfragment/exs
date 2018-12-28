---
title: "Intersection of rectangles"
date: 2018-12-28
tags: c
---

* TOC
{:toc}


## Description

Given two rectangles r1 and r2, return a rectangle representing
the intersection of r1 and r2. If the rectangles do not intersect,
return a rectangle with value 0 for both `width` and `height`.


## My solution

```c
#include <stdio.h>
#include <stdlib.h>


typedef struct {
  int x, y, width, height;
} rectangle;


// Helper functions.
int min(int a, int b) { return (a < b) ? a : b; }
int max(int a, int b) { return (a > b) ? a : b; } 


// Given a rectangle r, canonicalize its x and y coordinates
// by shifting negative values to non-negative values and adjusting
// the values of `width` and `height` accordingly.
rectangle canonicalize(rectangle r) {
  if (r.width < 0) {
    r.width = abs(r.width);
    r.x -= r.width;
  }
  if (r.height < 0) {
    r.height = abs(r.height);
    r.y -= r.height;
  }
  return r;
}


rectangle intersection(rectangle r1, rectangle r2) {
  r1 = canonicalize(r1);
  r2 = canonicalize(r2);

  int max_x1 = max(r1.x, r2.x);
  int max_y1 = max(r1.y, r2.y);
  int min_x2 = min(r1.x + r1.width,  r2.x + r2.width);
  int min_y2 = min(r1.y + r1.height, r2.y + r2.height);

  r1.x = max_x1;
  r1.y = max_y1;

  if (min_x2 < max_x1 || min_y2 < max_y1) {
    r1.width = 0;
    r1.height = 0;
  } else {
    r1.width = min_x2 - max_x1;
    r1.height = min_y2 - max_y1;
  }

  return r1;
}
```
