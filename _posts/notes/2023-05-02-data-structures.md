---
title: "CS Interview Prep Part 1: Data Structure Fundamentals"
date: 2023-05-02
permalink: /notes/2023/05/02/data-structures
tags:
--- 

# Dynamic Arrays
An array with dynamic size

Operations:
  - Appending: O(n) (must copy all old elements)
  - Lookup: O(1)

# Linked Lists
Multiple nodes consisting of a value and a pointer to the next node

Operations:
  - Appending: O(1)
  - Lookup: O(n)

# Hash tables
Keys mapped to locations of an array using a hashing function

Operations:
  - Appending: O(1)
  - Lookup: O(1) (could be slow with collisions)

# Binary Search Tree
Tree where all left children are smaller, right children are greater

Operations:
  - Appending: O(n)
  - Lookup: O(n)
  - Deleting: O(n)

# Heaps
Complete (every node has 2 children) binary tree

Max-heaps: all children are less than root

Min-heaps: all children are greater than root

# Graphs
Set of nodes and edges. Nodes contain values and edges point from node to node.

Can be directed or undirected edges (1 way or 2 way).

# Trees
Nonlinear data structure consisting of a root, children, and leaves
