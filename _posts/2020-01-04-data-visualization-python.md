---
title: "Data Visualization Using `matplotlib`, `seaborn`, and `pandas`"
date: 2020-01-04
permalink: /notes/2020/01/04/data-visualization-python
tags:
    - python    
    - - notebook
--- 

These are some simple visualization methods using python.
The most common libraries used for data visualization in python are
`matplotlib`, `seaborn`, and `pandas`.
So, we need to import them first: 

**In [1]:**

{% highlight python %}
import matplotlib as plt
import pandas as pd
import seaborn as sns
sns.set() # set styling to seaborn
{% endhighlight %}
 
Let's also load some generate some random data from `numpy`. 

**In [2]:**

{% highlight python %}
import numpy as np
values = np.random.rand(100,3)
df = pd.DataFrame(values, columns=['x', 'y', 'z'])
df.head()
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>x</th>
      <th>y</th>
      <th>z</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.721570</td>
      <td>0.149436</td>
      <td>0.911150</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.151494</td>
      <td>0.465817</td>
      <td>0.617048</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.668138</td>
      <td>0.149007</td>
      <td>0.805018</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.462841</td>
      <td>0.841917</td>
      <td>0.096166</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.355983</td>
      <td>0.677378</td>
      <td>0.030093</td>
    </tr>
  </tbody>
</table>
</div>


 
## Univariate Data 
 
### Quantitative Data 
 
For univariate quantitative data there are 4 options:
- Histograms/density plots 

**In [3]:**

{% highlight python %}
# Histograms
df['x'].hist()
{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x10e2f6550>



 
![png](https://i.imgur.com/4R5hZd9.png) 


**In [4]:**

{% highlight python %}
# Density Plots
df['x'].plot(kind='density')
{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x1103e87b8>



 
![png](https://i.imgur.com/WO66iRP.png) 

 
- Box plots/violin plots 

**In [5]:**

{% highlight python %}
# Box plots
sns.boxplot(x='x', data=df)
{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x1104fb9b0>



 
![png](https://i.imgur.com/cb5lmnJ.png) 


**In [6]:**

{% highlight python %}
# Violin plots
sns.violinplot(x='x', data=df)
{% endhighlight %}

    /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/scipy/stats/stats.py:1713: FutureWarning: Using a non-tuple sequence for multidimensional indexing is deprecated; use `arr[tuple(seq)]` instead of `arr[seq]`. In the future this will be interpreted as an array index, `arr[np.array(seq)]`, which will result either in an error or a different result.
      return np.add.reduce(sorted[indexer] * weights, axis=axis) / sumval





    <matplotlib.axes._subplots.AxesSubplot at 0x1105f4cc0>



 
![png](https://i.imgur.com/3k8EiyU.png) 

 
### Categorical Data 
 
Let's add some categorical data to our dataframe. 

**In [7]:**

{% highlight python %}
df['w'] = 0
df['w'] = df['w'].apply(lambda x: 'Apple' if np.random.random()<0.3 else 'Orange')
{% endhighlight %}
 
For univariate categorical data we usually just use bar plots: 

**In [8]:**

{% highlight python %}
sns.countplot(x='w', data=df)
{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x11071ce80>



 
![png](https://i.imgur.com/MJZDC7a.png) 

 
## Multivariate Data 
 
### Quantitative Data 
 
### Correlation
Often, we want to find **correlation** between certain quantitative data. For
example, if there is a column called `total_minutes_worked` and
`total_hours_worked`, we can imagine that the 2 would be correlated and one of
the columns is unnecessary.

To perform such a task, we use a heatmap 

**In [9]:**

{% highlight python %}
heatmap_df = df.drop('w', axis=1)
sns.heatmap(heatmap_df) # no correlation here
{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x1107b95c0>



 
![png](https://i.imgur.com/8cMwPHE.png) 

 
Otherwise, we usually can use either a `scatterplot` or `jointplot`. 

**In [10]:**

{% highlight python %}
# scatterplot
sns.scatterplot(x='x', y='y', data=df)
{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x110964240>



 
![png](https://i.imgur.com/u50aMbW.png) 


**In [11]:**

{% highlight python %}
# join plot
sns.jointplot(x='x', y='y', data=df)
{% endhighlight %}




    <seaborn.axisgrid.JointGrid at 0x110a19a58>



 

![png](https://i.imgur.com/n8lxr8R.png) 
