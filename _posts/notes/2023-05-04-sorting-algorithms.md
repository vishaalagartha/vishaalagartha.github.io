---
title: "CS Interview Prep Part 2: Sorting Algorithms"
date: 2023-05-04
permalink: /notes/2023/05/04/sorting-algorithms
tags:
--- 
 
# Sorting Algorithms 
 
### Merge Sort - O(nlogn)
* Recursively cut array in half and call merge sort on each half
* In merge step, insert elements in order 

**In [13]:**

{% highlight python %}
def merge(a1, a2):
    merged = []
    i=0
    j=0
    while i<len(a1) or j<len(a2):
        if i==len(a1):
            merged.append(a2[j])
            j+=1
        elif j==len(a2):
            merged.append(a1[i])
            i+=1
        elif a1[i]<a2[j]:
            merged.append(a1[i])
            i+=1
        else:
            merged.append(a2[j])
            j+=1
    return merged
            

def merge_sort(arr):
    if len(arr)==1: return arr
    mid = int(len(arr)/2)
    a1 = arr[:mid]
    a2 = arr[mid:]
    a1_sorted = merge_sort(a1)
    a2_sorted = merge_sort(a2)
    print(a1_sorted, a2_sorted)
    merged = merge(a1_sorted, a2_sorted)
    return merged
    
merge_sort([1, 3, 4, 3, 1])
{% endhighlight %}

    [1] [3]
    [3] [1]
    [4] [1, 3]
    [1, 3] [1, 3, 4]





    [1, 1, 3, 3, 4]


 
### Quick Sort - O(nlogn)
* Choose a partition index (could be first, random, last, etc)
* Put all elements smaller than partition to left, all elements greater than
partition to right
* Recurse on array from start to partition index and array from partition index
to end 

**In [23]:**

{% highlight python %}
def partition(arr,low,high): 
    i = low-1        
    pivot = arr[high]
  
    for j in range(low, high): 
        if   arr[j] <= pivot: 
            i = i+1 
            arr[i],arr[j] = arr[j],arr[i] 
  
    arr[i+1],arr[high] = arr[high],arr[i+1] 
    return i+1

def quickSort(arr,low,high): 
    if low < high: 
        pi = partition(arr,low,high) 
        quickSort(arr, low, pi-1) 
        quickSort(arr, pi+1, high)
    return arr
arr = [1, 4, 3, 6, 4, 7]
quickSort(arr, 0, len(arr)-1)
{% endhighlight %}




    [1, 3, 4, 4, 6, 7]


 
### Heap Sort - O(nlogn)
* Create a max heap using heapify

* Iteratively remove values from the max heap and add to end of array 

**In [43]:**

{% highlight python %}
def max_heapify(arr): 
    for i in range(0, len(arr)):
        largest = i
        l = 2*i+1
        r = 2*i+2
        
        if l<len(arr) and arr[largest]<arr[l]:
            largest = l
        if r<len(arr) and arr[largest]<arr[r]:
            largest = r
        if largest!=i:
            arr[i],arr[largest]=arr[largest],arr[i]
            max_heapify(arr)
    return arr

def heap_sort(arr):
    heap = max_heapify(arr)
    sorted_arr = []
    while len(heap)>0:
        sorted_arr.append(heap[0])
        heap = max_heapify(heap[1:])
    return sorted_arr[::-1]

arr = [21,1,45,78,3,5]
heap_sort(arr)
{% endhighlight %}




    [1, 3, 5, 21, 45, 78]


