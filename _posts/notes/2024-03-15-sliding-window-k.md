---
title: "Sliding Window Advanced Technique - At most k, k-1"
date: 2023-03-15
permalink: /notes/2024/03/15/sliding-window-k
tags:
    - leetcode
    - interview prep
    - sliding window
--- 

The sliding window is a common technique used to solve questions relating to *continuous* substrings or subarrays.

Typically, the problem is stated like one of the following:

> Find the length of the longest/shortest substring/subarray satisfying &lt;some condition&gt;.


> Count the number substrings/subarrays satisfying &lt;some condition&gt;.

It's trivial to come up with a `O(n^2)` solution to such problems - simply iterate overall subarrays/substrings. But the sliding window
can allow us to reduce the time complexity to `O(n)` by processing each element once as the window slides through the data.

This is a simple, but effective template to memorize that can be used for most cases:

{% highlight python %}
res = 0
i, j = 0, 0
while j < len(arr):
  # process arr[j] by updating a hashmap, count, etc...

  # decrease window size if invalid
  while not <condition>:
    # remove arr[i] from the window by updating a hashmap, count etc...
    i += 1
  
  # check if window is valid
  if <condition>:
    res = max/min(j - i + 1, res) # if finding min or max
    OR
    res += j - i + 1 # if finding count
  j += 1

# process remaining windows
while i < len(arr):
  if <condition>:
    res = max/min(j - i + 1, res) # if finding min or max
    OR
    res += j - i + 1 # if finding count
  i += 1
{% endhighlight %}

This should get you through some of the most common questions:

* [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
* [Frequency of the Most Frequent Element](https://leetcode.com/problems/frequency-of-the-most-frequent-element/)
* [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)
* [Longest Substring with at Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/)

The full list is available [here](https://leetcode.com/tag/sliding-window/).

But I want to discuss a technique slightly different where problems are stated like so:

> Count the number substrings/subarrays satisfying &lt;some condition *exactly* k&gt;.

Note that this does not say *at most* k or *at least* k, but *exactly* k.

This can be problematic since when executing the sliding window technique, we don't know what lies in the future and whether or not we should expand the window or not. Consider this simple question:

> Given a binary array nums and an integer goal, return the number of non-empty subarrays with a sum goal.

And as example consider

`nums = [1,0,1,0,1], goal = 2`

The answer here is 4:
```
[|1,0,1|,0,1]
[|1,0,1,0|,1]
[1|,0,1,0,1|]
[1,0,|1,0,1|]
```

So we can start off at `i, j = 0, 0` and continue expanding until `j=2` at which point our subarray will be `[1, 0, 1]`. In a typical sliding window approach, we would contract our window by incrementing `i`. But if we contract our window we risk potentially losing correct windows down the line.

The problem lies in the inclusion of 0's doesn't change our sum. We need a way to calculate *all* subarrays with sum equal to goal.

The trick to solving this is by transforming the problem into a different problem: Why don't we try and calculate the number of subarrays with sum **at most** equal to goal.

For the above array, we would have `[1]` (+1), `[1, 0]` (+2), `[1, 0, 1]` (+3), `[1, 0, 1, 0]` (+4), and `[0, 1, 0, 1]` (+4) for a total of 1 + 2 + 3 + 4 + 4 = 14. This can be achieved by simply using the above template and removing the condition where we check if the window is valid. After exiting the inner loop, the subarray and all the subarrays are valid. Since we're incrementing `j` in the outer loop, we accumulate all subarrays *ending at index j that have a sum at most k* during each execution of the outer loop.

Ok, but we're off by 10. How can we get rid of all the sums that are just below k? Easy enough. *Run the same algorithm for k-1`*.

This will accumulate the subarrays `[1]` (+1), `[1, 0]` (+2), `[0, 1]` (+2),  `[0, 1, 0]` (+3), `[0, 1]` (+2) for a total of 1 + 2 + 2 + 3 + 2 = 10.

So we've solved the problem of number of subarrays with at most k and the number of subarrays at most k - 1. The solution to our original question of number of subarrays is simply the difference of the two!

`exactly(k) = at_most(k) - at_most(k-1)`

It's a simple trick, but really useful once you start seeing it in other places. Here is the full code template for the question above:

{% highlight python %}
class Solution:
    def at_most(self, nums, goal):
        i, j = 0, 0
        s = 0
        res = 0
        while j < len(nums):
            s += nums[j]
            print(s)
            while i <= j and s > goal:
                s -= nums[i]
                i += 1
            res += j - i + 1
            j += 1
        return res
    
    def numSubarraysWithSum(self, nums: List[int], goal: int) -> int:
        at_most_goal = self.at_most(nums, goal)
        print(at_most_goal)
        at_most_goal_minus_1 = self.at_most(nums, goal - 1)
        return at_most_goal - at_most_goal_minus_1
{% endhighlight %}

So to recap:
* Use the 'At Most k, At Most k - 1' technique when searching for *exactly* k and we don't know if we should expand or contract the window
* At Most k subproblem is easier - simply count the number of subarrays that end at index `j`
* Subtract the two values to get *exactly* k

Here are some problems that can be solved using this technique to practice on. I will add more to the list as I encounter them:
* [Binary Subarrays With Sum K](https://leetcode.com/problems/binary-subarrays-with-sum/)
* [Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/)