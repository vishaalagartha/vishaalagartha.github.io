---
title: "Basics of Bitmasking - From Backtracking to Dynamic Programming"
date: 2023-04-02
permalink: /notes/2024/04/02/bitmasking-basics
tags:
    - leetcode
    - bitmask
    - dynamic programming
--- 

## Introduction
When attempting to achieve a performant solution to a problem that seems fairly brute-forcey, we often turn to dynamic programming. But DP necessecitates us to maintain some sort of *state* in order efficiently cache intermediate solutions.

The problem arises when our state that needs to be maintained appears uncacheable. Take the problem [Maximum AND Sum of Array](https://leetcode.com/problems/maximum-and-sum-of-array/) for example.

We're given a number `numSlots` array `nums` of length `numSlots*2`. Each slot has 2 open positions and we have to place each number in `nums` inside a slot. The AND sum of the result is the sum of the number in the slot and slot number for all numbers in `nums`. For example if we have:

`nums = [1,2,3,4,5,6], numSlots = 3`

The maximum AND sum achievable is 9 via the following configuration

`Slot 1: [1, 4] | Slot 2: [3, 5] | Slot 3: [2, 6]`

Because

`(1 & 1) + (1 & 4) + (2 & 3) + (2 & 5) + (3 & 2) + (3 & 6) = 9`

## Diagnosis

Ok so we see there is some feature of bits here. Maybe we can optimize by precomputing the bits for each slot and number and pairing them in an efficient manner? But let's look at the time constraints:

```
1 <= numSlots <= 9
1 <= nums[i] <= 15
```

Ok, so this **screams** brute force. With such a small number of `numSlots`, we can probably try all possible combinations and return the result. 

## Brute force
Let's take the backtracking brute force method via recursion. At each recursive we choose the first element in nums and try pairing it with another. We then create a new array `newNums` and pass it to the next level. When we've exhausted all numbers, we return 0 as our base case:

{% highlight python %}
def maximumANDSum(self, nums: List[int], numSlots: int) -> int:
    def dfs(nums):
        if not nums:
            return 0
        res = 0
        for i in range(len(slots)):
            if slots[i] < 2:
                slots[i] += 1
                res = max(res, dfs(nums[1:]) + (nums[0] & (i + 1)))
                slots[i] -= 1
        return res
    slots = [0] * numSlots
    return dfs(nums)
{% endhighlight %}

Ok, so this works, but it results in TLE. How can we go about optimizing this into DP?

## Managing state

The key to transforming this problem is to understand, how can we manage state? Right now, if we put `nums[0]` and `nums[1]` into slot 0, we're performing a lot of work to solve all the subproblems and letting our work go to waste by not caching them.

Somehow, we need to maintain how many slots are open in each slot. We know there are at most 2 open positions, so why not encode it by using a `0` for open and `1` for taken? For example if `numSlots=3`, `000000` indicates that there are no spots taken and `000001` indicates one slot in slot 1 in taken and `000011` indicates *both* slots are taken in slot 1.

Note that we *could* encode it as a string, but we will opt to use the integer representation of the binary string to expedite our checking process.

Now how can we check whether a slot is taken? We need to basically see if slots indices `2*i` and `2*i + 1` are both taken. If not we can take the slot and update the state accordingly.

Now this is where bitmasking shines. Rather than looping through a binary string and checking values and updating them, we can use bit operations to quickly check if any index is 1 or 0 and update the current state to have a 1 if we choose to take the slot.

To check if a slot is taken, we can left shift 1 by a chosen number of indices and use the `&` operation. If the `&` operation results in 0, we know the slot is not taken.

To update the current state, we can left shift 1 by the same number of indices and use the `|` operation. The `|` operation will update a position that was 0 with a 1.

To check:

`mask & 1<<i == 0`

To update:

`mask |= 1<<i`

We now pass the updated state into the next level of recursion, incrementing the index we are trying to place by 1.

Here is commented version of the code:

{% highlight python %}
@lru_cache(None)
def dfs(i, mask):
    if i == len(nums):
        return 0
    res = 0
    # Check which slots are open
    for j in range(0, 2*numSlots, 2):
        # slot number is (j // 2 + 1)
        m1 = 1<<j
        m2 = 1<<(j + 1)
        # Check if first slot is open
        if mask & m1 == 0:
            res = max(res, dfs(i + 1, mask | m1) + (nums[i] & (j // 2 + 1)))
        # Check if second slot is open
        elif mask & m2 == 0:
            res = max(res, dfs(i + 1, mask | m2) + (nums[i] & (j // 2 + 1)))
    return res


return dfs(0, 0)
{% endhighlight %}

Easy enough! It's practically the same code except we're maintaining some state. Additionally note how the initial state is 0, which corresponds to the binary string `0000....0000`.

I hope this helps provide some insight on how to use bitmasking to transform your brute force approach into a dp solution!

I encourage you to go through [this list](https://leetcode.com/tag/bitmask/) to get some practice on this technique.
