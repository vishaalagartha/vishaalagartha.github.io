---
title: "D3-Squared: Definitive guide to D3 Data joins"
date: 2019-03-20
permalink: /notes/2019/03/20/data-joins
--- 

* NOTE: this is a simplified explanation. For a deeper, in-depth explanation with code and interactive tutorials, I recommend you check out my [bl.ock](https://bl.ocks.org/vishaalagartha/8d7339ebf4cb31ae20ed5ddb03b9ad6e) on the topic!

D3's model for binding data to DOM elements can often seem tedious and complicated. But, I actually think people overcomplicate it. Here is my definitive guide:

There are 4 key steps that are the foundation. I'm going to lay them all out.

### 1) Create the data join
First, we bind a DOM element to some data. This is performed using the `.selectAll()` and `.data()` functions. Here is an example:

```javascript
let data = [1, 2, 3]
let dataJoin = domElement.selectAll('div').data(data)
```

Now, we have created a data join consisting of 3 `div`'s, one for each element in the array `data`.

### 2) Create

Next, we create elements by entering and appending to the data join. This is performed via the conveniently named `.enter()` and `.append()` functions.

```javascript
dataJoin.enter().append('div')
```

Now, we have created 3 `div`'s that will show up. We can also add attributes, styles, and other doo-dads here. Note that we have access to each element in data here and these attributes can be a function of the data element. For example, if we were creating circles we could say:

```javascript
let data = [{fill: 'red'}, {fill: 'green'}, {fill: 'blue'}]
let dataJoin = domElement.selectAll('circle').data(data)
dataJoin.enter().append('div')
        .attr('class', 'myCircleClass')
        .attr('r', 30)
        .attr('cx', 10)
        .attr('cy', 10)
        .style('fill', d => d.fill) // we can access each element in the array here
```

### 3) Remove

To remove extra elements, we can use the `.exit()` and `.remove()` functions. So, if we redefine the above data to be

```javascript
data = [{fill: 'green'}, {fill: 'blue'}]
```

and perform the remove.

```
dataJoin.exit().remove()
```

There would only be 2 circles. But, note that D3 does not distinguish between individual elements and will simply match array sizes. Hence, the first 2 elements of the original data join will remain, but the third will be removed, manifesting in red and green circles even though `data` has been redefined to contain green and blue circles.

### 4) Update
To fix the above problem, one must update the data join prior to removing extraneous elements and after creating new elements. So, in between, we simply update by adding the relevant attributes, classes, etc.

### Create, Update, Remove - The Full Cycle

So now, we can demonstrate the full cycle of D3:

```javascript
let dataJoin = domElement.selectAll('div').data(data)
// Create
dataJoin.enter().append('div')
        .attr('class', 'create')

// Update
dataJoin.attr('class', 'update')

// Remove
dataJoin.exit().remove()
```

### An aside - Merge
D3 v5 introduces a new idea of merging both the existing and newly created elements and performing updates on both. Otherwise, we will only update existing elements. This is achieved via the `.merge()` function. You simply apply the function after creating the new elements and subsequent updates will apply to both old and new elements.

```javascript
dataJoin.enter().append('div')
        .attr('class', 'create')
        .merge(dataJoin)
        .attr('class', 'update')
```
