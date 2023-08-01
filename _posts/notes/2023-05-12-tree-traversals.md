---
title: "CS Interview Prep Part 3: Tree Traversals"
date: 2023-05-12
permalink: /notes/2023/05/12/tree-traversals
tags:
--- 
# Binary Search Tree Traversals 

**In [30]:**

{% highlight python %}
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None
        
root = TreeNode(1)
left = TreeNode(2)
right = TreeNode(3)
root.left = left
root.right = right
left_left = TreeNode(4)
left_right = TreeNode(5)
left.left = left_left
left.right = left_right
{% endhighlight %}
 
Consider the following BST: 

**In [5]:**

{% highlight python %}
%%capture
'''
        1
      /   \
     2     3
   /   \
 4       5
'''
{% endhighlight %}
 
## In Order Traversal

Traverses the BST **in-order** using the following pattern: Left, Root, Right

In the above example: 4, 2, 5, 1, 3 

**In [9]:**

{% highlight python %}
def in_order_recursive(root):
    if root.left:
        in_order_recursive(root.left)
    print(root.val)
    if root.right:
        in_order_recursive(root.right)
in_order_recursive(root)
{% endhighlight %}

    4
    2
    5
    1
    3


**In [12]:**

{% highlight python %}
def in_order_iterative(root):
    n = root
    stack = []
    while n or stack:
        # go to left most element in subtree
        if n:
            stack.append(n)
            n = n.left
        # visit top of stack and then traverse right subtree
        elif stack:
            n = stack.pop()
            print(n.val)
            n = n.right
in_order_iterative(root)
{% endhighlight %}

    4
    2
    5
    1
    3

 
## Pre Order Traversal

Traverses the BST using the following pattern: Root, Left, Right

In the above example: 1, 2, 4, 5, 3 

**In [14]:**

{% highlight python %}
def pre_order_recursive(root):
    print(root.val)
    if root.left:
        pre_order_recursive(root.left)
    if root.right:
        pre_order_recursive(root.right)
pre_order_recursive(root)
{% endhighlight %}

    1
    2
    4
    5
    3


**In [17]:**

{% highlight python %}
def pre_order_iterative(root):
    stack = [root]
    while stack:
        n = stack.pop()
        print(n.val)
        if n.right:
            stack.append(n.right)
        if n.left:
            stack.append(n.left)
pre_order_iterative(root)
{% endhighlight %}

    1
    2
    4
    5
    3

 
## Post Order Traversal

Traverses the BST using the following pattern: Left, Right, Root

In the above example: 4, 5, 2, 3, 1 

**In [18]:**

{% highlight python %}
def post_order_recursive(root):
    if root.left:
        post_order_recursive(root.left)
    if root.right:
        post_order_recursive(root.right)
    print(root.val)

post_order_recursive(root)
{% endhighlight %}

    4
    5
    2
    3
    1


**In [32]:**

{% highlight python %}
def post_order_iterative(root):
    stack1 = [root]
    stack2 = []
    while stack1:
        n = stack1.pop()
        stack2.append(n)
        if n.left:
            stack1.append(n.left)
        if n.right:
            stack1.append(n.right)

    while stack2:
        print(stack2.pop().val)
post_order_iterative(root)
{% endhighlight %}

    4
    5
    2
    3
    1

 
## Level Order Traversal

Traverses the BST level by level (Breadth First Search)

In the above example: 1, 2, 3, 4, 5 

**In [34]:**

{% highlight python %}
def level_order_recursive(nodes):
    if len(nodes)==0: return
    l = []
    for n in nodes:
        print(n.val)
        if n.left: l.append(n.left)
        if n.right: l.append(n.right)
    level_order_recursive(l)

level_order_recursive([root])
{% endhighlight %}

    1
    2
    3
    4
    5


**In [35]:**

{% highlight python %}
def level_order_iterative(root):
    queue = [root]
    while queue:
        n = queue[0]
        queue = queue[1:]
        print(n.val)
        if n.left: queue.append(n.left)
        if n.right: queue.append(n.right)
            
level_order_iterative(root)
{% endhighlight %}

    1
    2
    3
    4
    5


**In [None]:**

{% highlight python %}

{% endhighlight %}
