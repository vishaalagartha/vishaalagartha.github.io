---
title: "Sum of distances in array"
date: 2023-03-18
permalink: /notes/2024/03/18/sum-of-distances
tags:
    - leetcode
    - interview prep
    - prefix sum
--- 

I ran into a similar pattern Leetcode problem over the past 3 days and decided it was a common enough pattern that I should make a post about it.

For reference here are the two problems:
* [Minimum Operations to Make All Array Elements Equal](https://leetcode.com/problems/minimum-operations-to-make-all-array-elements-equal/)
* [Sum of Distances](https://leetcode.com/problems/sum-of-distances/submissions/1207749137/)

I won't go into the nuances of each problem, but after transforming each to a certain degree, I arrived at the following problem:

> Given a list of numbers. Find the sum of the absolute differences between a given value and every number in the list.

Consider the following example:

```
numbers = [3,1,6,8]
value = 5

result = abs(3-5) + abs(1-5) + abs(6-5) + abs(8-5) = 2 + 4 + 1 + 3 = 10
```

We can do this trivially by simply looping through the list if we just need to perform this action for a single number. But what if we need to do it for multiple? It transforms the problem from a O(n) solution to a potentially O(n * k) solution for k numbers.


So how can we go about doing this? Let's rephrase the problem like so: **the cost of bringing up all the numbers less than value up to value + the cost of bringing all the numbers greater than value down to value**.

This changes our mentality - we clearly need to find the number of values less than and greater than our given value. We can do this relatively quickly in a two step process:

1) Sort the elements

2) Perform binary search to find the left insertion point of the value (or `bisect.bisect_left` in Python) - this is the number of values less than the chosen number

3) Perform binary search to find the right insertion point of the value (or `bisect.bisect_right` in Python) - the length of the array minues this index is the number of values greater than the chosen number

Using the above example we 

1) Sort to obtain `[1,3,6,8]`

2) Find the left insertion point to be 2 --> there are 2 values less than `5`.

3) Find the right insertion point to be 2 --> there are 4 - 2 = 2 values greater than `5`.

Now, let's rephrase it *again*. We know there are 2 values less than 5. To bring these values up to 5 is equivalent to bringing their cumulative sum up to 5*2 = 10. Or:

`(5-3) + (5-1) = (5 + 5) - (3 + 1)`

Similarly, to bring the later values down to 5 is equivalent to bringing their cumulative sum down to 5*2 = 10:

`(8-5) + (6-5) = (8 + 6) - (5 + 5)`

Those two summations should look familiar. We are simply aggregating values in succession starting from left to right and right to left - the prefix and suffix sums of the arrays.

So let's get those:

`prefix = [0, 1, 4, 10, 18], suffix = [18, 17, 14, 8, 0]`

Now we can use these indices to find the two distances. For example for the value 5:

The left insertion point is 2. There are 2 numbers less than 5. The prefix sum value at this index is 4. So we take (5*2) - 4 = 6.

The right insertion point is also 2. There are 2 numbers greater than 5. The suffix sum at this index is 14. So we take 14 - (5*2) = 4.

To to bring the lower 2 values up, it takes 6 and to bring the two upper values down, it takes 4. So the solution is 4.

Let's put this more explicitly into code (note this is literally copy and pasted from my solution for [Minimum Operations to Make All Array Elements Equal](https://leetcode.com/problems/minimum-operations-to-make-all-array-elements-equal/)):

{% highlight python %}
res = []
for q in queries:
    idx_left = bisect.bisect_left(nums, q)
    nums_left = idx_left
    sum_left = prefix[idx_left]
    left_distance = (q * nums_left) - sum_left
    idx_right = bisect.bisect_right(nums, q)
    nums_right = len(nums) - idx_right
    sum_right = suffix[idx_right]
    right_distance = sum_right - (q * nums_right) 
    res.append(left_distance + right_distance)
return res
{% endhighlight %}

I hope you find this technique useful!