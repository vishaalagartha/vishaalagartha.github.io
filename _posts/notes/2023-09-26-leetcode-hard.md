---
title: "Leetcode Hard Problem Cheatsheet"
date: 2023-09-26
permalink: /notes/2023/09/26/leetcode-hard
--- 

A quick guide to some of the most popular LeetCode Hard questions

### Candy
- Left to right array - check if prev > curr
- Right to left array - check if next > curr
- Add to total max of both


### Largest Rectangle in histogram
- Maintain monotonic stack with index, height
- Keep adding to stack as long as heights are increasing
- While height decreases, pop from stack and recompute max area


### Trapping Rain Water
- Keep left to right max height array and right to left max height array
- Take min of both and subtract current index


### Sliding Window Median
- Min heap maintains lower half of data
- Max heap maintains top half of data
- When sliding window, check which heap element is in and pop (don’t need to actually pop element)


### Median from data stream
- Min heap maintains lower half of data
- Max heap maintains top half of data


### Longest Valid Parenthesis
- Maintain stack with indices of open parenthesis
- When closed parenthesis
    - Pop from stack
        - If stack still has value - subtract index - top of stack
        - Otherwise - put closed parenthesis index on stack
- Stack initialized with -1


### Sudoku Solver - backtrack

### N-Queens - backtrack

### Permutation Sequence - recursion by adding 1 element at a time to string

### Binary Search Tree Serialization
- Serialize as preorder traversal (DFS)
- Deserialize with global variable idx using preorder traversal (DFS)


### Binary Tree Maximum Sum
- Store maxSum with split at current node as global variable
- Compute maxSum without splitting at current node
- Return without splitting
### Wordbreak II - DFS with a Trie


### Max Points along line
- Store slopes for each line
- Don’t forget about duplicate points


### Skyline
- Sort array with points and start or end
- Maintain max heap initialized with 0
- If start
    - If height exceeds max heap —> add x, h
    - Add height to max heap
- If end
    - Remove from heap
    - heapify
    - If height exceeds max heap —> add x, max heap to res


### Maximal Rectangle
- Use maximal rectangle in a histogram algorithm
- Add rows that stack on one another


### Median of 2 sorted Arrays
- Partition 2 arrays into 2 chunks 
- Ensure A_left <= B_right, B_left <= A_right
- Gotchas:
    - Out of bounds checking with float(inf), float(-inf)
    - Swap to make A the larger of 2
    - Index subtract by 2 (due to 0 indexing)


### Number of elements smaller than self
- Maintain sorted array
- Count number of elements to left of bisect left index


### LRU/LFU Cache
- Doubly Linked List
- Hashmap with frequency
- Hashmap with mapping to node
