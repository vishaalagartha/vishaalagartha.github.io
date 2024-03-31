---
title: "Chebyshev Distance - How to Transform 2d problems to 1d"
date: 2023-03-31
permalink: /notes/2024/03/31/chebyshev-distance
tags:
    - leetcode
    - geometry
--- 

## Introduction

This won't be a super long post as it was mostly inspired by this week's [Weekly Content](https://leetcode.com/contest/weekly-contest-391/) where I got stuck on question 4 due to not knowing this simple trick. The method is fairly well documented in other locations, so I want this to serve as a reminder to myself more than anything about this technique.

## Problem Statement

The problem can be boiled down to:

> Find the maximum Manhattan distance between 2 points on the coordinate plane.

Note that Manhattan distance is defined as `abs(x1-x2) + abs(y1-y2)` for two points `(x1, y1)` and `(x2, y2)`.

The brute force solution is to simply through all pairs of points and maintain the maximum. This is obviously `O(n^2)` and won't suffice in most scenarios. This trick will allow us to solve the problem in `O(nlogn)`

## Trick

I'll jump right to the trick since it's nearly impossible to derive it without seeing the technique first. The trick is transform all points `(x, y)` into `(x+y, x-y)` or `(s, d)`. Then find the maximum and minimum value between all points for these new data point by sorting (hence the `O(nlogn)`). Then simply take the difference between the maximum and minimum for both all `s` values and all `d` values. The answer will be the maximum of these. More explicitly:

`max(max(s) - min(s), max(d) - min(d))`

 Why does this work?

The Manhattan distance between two points can be written as

`abs(x1-x2) + abs(y1-y2)`

This is equivalent to 

`max(x1-x2-y1+y2, -x1 + x2 + y1 – y2, -x1 + x2 – y1 + y2, x1 – x2 + y1 – y2)`

Which can be rearranged as

`max((x1 – y1) – (x2 – y2), (-x1 + y1) – (-x2 + y2), (-x1 – y1) – (-x2 – y2), (x1 + y1) – (x2 + y2))`

The first two values are the differences between `x` and `y` (i.e. `d`) and the latter two are the sums between the two `x` and `y` (i.e. `s`).

In other words, the answer is one of the maximum of difference between the sums and differences.

## Code

Now I know there isn't very much theory here, so I think the best approach is to provide a template and practice. This is some skeleton code to find the maximum distance that can be regurgitated for most cases:

{% highlight python %}
s = [x + y for x, y in points]
d = [x - y for x, y in points]
max_dist = max(max(s) - min(s), max(d) - min(d))       
{% endhighlight %}

## Practice Problems

Here are some use cases to practice this technique:

* [Minimize Manhattan Distance](https://leetcode.com/problems/minimize-manhattan-distances)
* [Maximum Absolute Value Expression](https://leetcode.com/problems/maximum-of-absolute-value-expression)
* [Minimum Time Visiting All Points](https://leetcode.com/problems/minimum-time-visiting-all-points/description/)