---
title: "CS Interview Prep Part 7: System Design Algorithms"
date: 2020-05-27
permalink: /notes/2020/05/27/system-design-algorithms
tags:
--- 
# System Design Algorithms

## Bloom Filter
Checks whether an element is in a set quickly and with minimum resources.

Use Case: Check whether a URL is in a set of already visited URLs in a web crawler

## Geohash
Location based search results

Use Case: Dating app that allows users to filter by vicinity

## Leaky Bucket/Token Bucket
Limit the amount of traffic in a certain network

Use Case: API rate limiter to prevent user from spamming a server

## Lossy Counting
Check if items in a data stream exceeds user given count

Use Case: Find ranking in a system

## Quadtree/RTree
Find nearby interest points

Use Case: Similar to Geohash

## Ray Casting
Check if point is inside polygon

Use Case: Check if point is in country

## Reverse Index
Storing data based on keywords

Use Case: Storing a web crawler's data to handle user queries

## Trie Algorithm
Retrieve data based on prefix

Use Case: Autocomplete system
