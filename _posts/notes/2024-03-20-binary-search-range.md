---
title: "Beyond Binary Search - Searching for Ranges"
date: 2023-03-20
permalink: /notes/2024/03/20/binary-search-range
tags:
    - leetcode
    - interview prep
    - binary search
--- 

## Introduction

With libraries like `bisect` and simple memorization, I think people often forget to look into binary search into a little more depth than the standard template:

{% highlight python %}

def bisect_left(arr):
    def condition(y):
        ...

    lo, hi = 0, len(arr) - 1
    while lo < hi:
        mid = (hi + lo) // 2
        if condition(mid):
            lo = mid + 1
        else:
            hi = mid
    return lo

{% endhighlight %}

### Basic Application

Binary search can seem like a simple algorithm with a single use case:

> Given a sorted list, find the first 'bad' element.

This type of question can come in various styles. Some common ones I've encountered are [First Bad Version](https://leetcode.com/problems/first-bad-version/) or [First Missing Positive in a Sorted Array](https://www.reddit.com/r/leetcode/comments/1biqjob/the_interviewer_asked_me_to_optimize_this_but_i/).

### Slightly Advance Application

A slightly more nuanced application arises when **the target is not readily available for us**. These problems can be trickier to diagnose and we may turn to DFS or dynamic programming only to result in TLE.

But, I'd encourage you to ask yourself whether there is some kind of *monoticity* we can use. More explicitly, if `condition(x)` is true, is `condition(x+1)` also true? Let's dive deeper with an example.

[Capacity To Ship Packages Within d Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/description/) asks us to find "the least weight capacity of the ship that will result in all the packages on the conveyor belt being shipped within `d` days."

In other words, given an array of weights, we want to find the minimum capacity so that we can ship all the weights in `d` days. Please read the description for a lengthier explanation with examples.

But the idea is, if we can say that we **cannot ship all the weights in `d` days with a capacity of `x`, we for sure can guarantee that we cannot ship all the weights in `d` days with a capacity of `x + 1`**. This is monoticity that I mean.

{% highlight python %}
def shipWithinDays(self, weights: List[int], days: int) -> int:
    def canShip(capacity):
        count = 1
        curr = 0
        for w in weights:
            if curr + w <= capacity:
                curr += w
            else:
                curr = w
                count += 1
        return count <= days
    
    maxVal = max(weights)
    lo = maxVal
    hi = sum(weights)
    
    while lo < hi:
        mid = (lo + hi) // 2
        # Check if we can ship in this many days
        # If we can, then shorten window to second half
        # If we cannot, then shorten window to first half
        if not canShip(mid):
            lo = mid + 1
        else:
            hi = mid
    return lo
{% endhighlight %}

In this example, we are not given the search range, but we must deduce that the search space is limited and the search is monotonic. Other classic problems of this type are [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/description/) or [House Robber IV](https://leetcode.com/problems/house-robber-iv).

### More Advanced Application

This is the next level of binary search that I recently encountered in an Online Assessment. The goal of the problem was to find the number of points on a number line that satisfy a condition. In this case, we are asked to find range of values that are valid with no clear values to test. More explicitly:

> Given an array of centers and a distance `d`. Find the number of valid points on the number line where you can travel to each center and back with a distance under `d`.

For example, given the `centers = [2, 0, 3, -4]`, `d=22`.

If try `x=-3`, we get `d = 2 * abs(2 - (-3)) + 2 * abs(0 - (-3)) + 2 * abs(3 - (-3)) + 2 * abs(-4 - (-3)) = 10 + 6 + 12 + 2 = 30 > 22`. So -3 is invalid.

But if try `x=0`, we get `d = 2 * abs(2 - 0) + 2 * abs(0 - 0) + 2 * abs(3 - 0) + 2 * abs(-4 - 0) = 4 + 0 + 6 + 8 = 18 < 22`. So 0 is valid.

In fact, we can see that the valid points are `-1, 0, 1, 2, 3, 4` and the answer is `4 - (-1) = 5`.

Evidently, there is a lower bound that works and an upper bound that works. Our goal should be to find the lower bound and the upper bound and return the difference!

Finding the lower bound is rather trivial - simply use the same template as above for the packages problem. In other words, search for *the minimum point on the line where we can reach all the centers and back within or equal to d*.

To find the upper bound, we flip the binary search algorithm to **find the maximum** value that satisfies the condition. In other words, `bisect_right`.

What do I mean by this? Let's say you're given the array `[0, 1, 1, 1, 2]` and are looking to insert the number `1`. Where should you insert it? At index 1 or index 4? It could go at either location, so both are valid answers. `bisect_left` allows us to find the minimum value that satisfies the condition (i.e. index 1) and `bisect_right` allows us to find the maximum value that satisfies the condition.

So how does `bisect_right` look in code?

{% highlight python %}

def bisect_right(arr):
    def condition(y):
        ...

    lo, hi = 0, len(arr) - 1
    while lo < hi:
        mid = (hi + lo) // 2
        if condition(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo

{% endhighlight %}

Here, if the condition is true, we say that we should search the bottom half of the array. If the condition is false, we should search the top half of the array.

Now with both methods of bisection, we can:

1) Search in an array given explicit values (simple case)

2) Search for the minimum value satisfying a monotonic condition (slightly advanced case)

3) Find the range of *all values that satisfy a monotonic condition* (more advanced case)

With that, I'll leave my code for the final problem below for your reading. I hope you learned something!

{% highlight python %}
center = [2, 0, 3, -4]
d = 22

def distance(x):
  res = 0
  for c in center:
    res += 2 * (abs(c - x))
  return res

center.sort()
i = center[0] - d // 2
j = center[-1] + d // 2
# Get left insertion point
while i < j:
  mid = (i + j) // 2
  dist = distance(mid)
  if dist <= d:
    i = mid + 1
  else:
    j = mid

start = i

# Get right interstion point
i = center[0] - d // 2
j = center[-1] + d // 2
# Get left insertion point
while i < j:
  mid = (i + j) // 2
  dist = distance(mid)
  if dist <= d:
    j = mid
  else:
    i = mid + 1
end = i
print(start, end) # 4, -1
print(max(start - end, 0)) # 5

{% endhighlight %}
