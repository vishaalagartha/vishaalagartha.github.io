---
title: "Analyzing Voter Fatigue"
date: 2020-01-29
permalink: /projects/2020/01/29/voter-fatigue
tags:
    - python
    - notebook
--- 
 
## Introduction

Legacies in the NBA are defined by 2 items more than anything:
* Championships
* Awards

Whereas championships are earned purely objectively through competition, awards
present a more *subjective* element. As a result, the number of awards a player
receives is more prone to bias.

A landmark moment to illustrate this pattern was in 2011 when Derrick Rose won
the MVP over Lebron James.

Lebron James was coming off of an incredible year in Miami with just around 27
points, 7.5 rebounds, and 7 assists per game that season with a field goal
percentage of 51%. In comparison to Rose’s 25 points, 4 rebounds, and 8 assists
with a field goal percentage of 45%, Lebron had the better statistics for that
season. But, Lebron had just come off of back-to-back MVP's, so who would want
to vote for him again.

This tendency to favor the new candidate over the previous year's winner is
known as **voter fatigue**. It is a pattern that occurs for individual awards
such as the MVP, DPOY, MIP, SMOY, and COY.

So, how much does voter fatigue matter for each of the individual awards? This
project aims to investigate how much *better* a winner of each individual award
must be the following year to repeat. 
 
## Methodology

### Measuring A Candidate's Success
For each award we choose the most relevant statistic that determines the award.

For the MVP, we choose Win Shares (WS) since the MVP should be determined by the
player who contributes most to the team's success.

For DPOY, we choose Defensive Win Shares (DFS) since the DPOY should be
determined by the player who contributes the most to the team's success
*exclusively on the defensive end*.

For MIP, we choose the difference between the player's PER the year they were
selected and the player's PER the year prior.

For SMOY, we choose the player's PER that year.

For COY, we choose the difference between the number of wins the team won that
year and the number of wins the team was predicted to win ([Pythagorean
Wins](https://www.basketball-reference.com/about/glossary.html) on Basketball
Reference).

To summarize:

|Award|Metric|
|--|--|
|MVP|WS|
|DPOY|DWS|
|MIP|CURR_PER-PREV_PER|
|SMOY|PER|
|COY|W-PW|

### Measuring A Candidate's Award Voting

For MVP, DPOY, MIP, and SMOY, we aggregate the the share of the points that the
player received during voting from the years 1988 onward since award statistics
were first released in 1988.

Since the points share is always scaled between 0.0 and 1.0, we will be able to
compare between awards appropriately.

Unfortunately, award statistics were only released in 2016 for COY, so our
dataset will be much smaller. 
 
### Computing Voter Fatigue

For each award, we compute the difference between the points share the candidate
received the following year. Next, we compute the difference between the
relevant scaled statistic.

For example (note that WS is not scaled in this example for clarity):

Michael Jordan won the MVP in 1988 with a WS of 21.2 and a PTS_SHARE of 0.831.
In 1989, he lost to Magic Johnson, with a WS of 19.8 and a PTS_SHARE of 0.704.

So the two data points we compute are 21.2-19.8 = 0.4, and 0.831-0.704=0.127

We then plot the difference in the Points Share as a function of the statistic's
difference and fit a regression line.

Finally, we find the x-intercept of this regression line.

This x-intercept measures how much better the candidate must perform **if they
were to maintain their performance from their previous year AND receive the same
points share**.

Using this value, we can quantify voter fatigue for each award.

Let's begin! 

**In [1]:**

{% highlight python %}
import pandas as pd
import seaborn as sns
from matplotlib import pyplot as plt
import numpy as np
sns.set()
{% endhighlight %}
 
### MVP Award 

**In [2]:**

{% highlight python %}
df = pd.read_csv('mvp_data.csv')
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
      <th>RANK</th>
      <th>PLAYER</th>
      <th>AGE</th>
      <th>TEAM</th>
      <th>FIRST_PLACE_VOTES</th>
      <th>TOTAL_POINTS_WON</th>
      <th>TOTAL_POINTS_POSSIBLE</th>
      <th>POINTS_SHARE</th>
      <th>G</th>
      <th>MP</th>
      <th>...</th>
      <th>TRB</th>
      <th>AST</th>
      <th>STL</th>
      <th>BLK</th>
      <th>FG%</th>
      <th>3P%</th>
      <th>FT%</th>
      <th>WS</th>
      <th>WS/48</th>
      <th>SEASON</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Michael Jordan</td>
      <td>24</td>
      <td>CHI</td>
      <td>47.0</td>
      <td>665.0</td>
      <td>800</td>
      <td>0.831</td>
      <td>82</td>
      <td>40.4</td>
      <td>...</td>
      <td>5.5</td>
      <td>5.9</td>
      <td>3.2</td>
      <td>1.6</td>
      <td>0.535</td>
      <td>0.132</td>
      <td>0.841</td>
      <td>21.2</td>
      <td>0.308</td>
      <td>1988</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Larry Bird</td>
      <td>31</td>
      <td>BOS</td>
      <td>16.0</td>
      <td>527.0</td>
      <td>800</td>
      <td>0.659</td>
      <td>76</td>
      <td>39.0</td>
      <td>...</td>
      <td>9.3</td>
      <td>6.1</td>
      <td>1.6</td>
      <td>0.8</td>
      <td>0.527</td>
      <td>0.414</td>
      <td>0.916</td>
      <td>15.0</td>
      <td>0.243</td>
      <td>1988</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Magic Johnson</td>
      <td>28</td>
      <td>LAL</td>
      <td>16.0</td>
      <td>508.0</td>
      <td>800</td>
      <td>0.635</td>
      <td>72</td>
      <td>36.6</td>
      <td>...</td>
      <td>6.2</td>
      <td>11.9</td>
      <td>1.6</td>
      <td>0.2</td>
      <td>0.492</td>
      <td>0.196</td>
      <td>0.853</td>
      <td>10.9</td>
      <td>0.199</td>
      <td>1988</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Charles Barkley</td>
      <td>24</td>
      <td>PHI</td>
      <td>1.0</td>
      <td>109.0</td>
      <td>800</td>
      <td>0.136</td>
      <td>80</td>
      <td>39.6</td>
      <td>...</td>
      <td>11.9</td>
      <td>3.2</td>
      <td>1.3</td>
      <td>1.3</td>
      <td>0.587</td>
      <td>0.280</td>
      <td>0.751</td>
      <td>16.7</td>
      <td>0.253</td>
      <td>1988</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Clyde Drexler</td>
      <td>25</td>
      <td>POR</td>
      <td>0.0</td>
      <td>86.0</td>
      <td>800</td>
      <td>0.108</td>
      <td>81</td>
      <td>37.8</td>
      <td>...</td>
      <td>6.6</td>
      <td>5.8</td>
      <td>2.5</td>
      <td>0.6</td>
      <td>0.506</td>
      <td>0.212</td>
      <td>0.811</td>
      <td>13.2</td>
      <td>0.207</td>
      <td>1988</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>



**In [3]:**

{% highlight python %}
l1 = 'WS Year After MVP - WS MVP Year'
l2 = 'PS Year After MVP - PS MVP Year'

chart_df = pd.DataFrame(columns=[l1, l2, 'Player'])
for y in df['SEASON'].unique():
    if y==1988: continue
    prev_y = y-1
    prev_y_df = df[df['SEASON']==prev_y]
    prev_winner = prev_y_df[prev_y_df['RANK']=='1']
    y_df = df[df['SEASON']==y]
    prev_winner_name = prev_winner['PLAYER'].values[0]
    prev_winner_now = y_df[y_df['PLAYER']==prev_winner_name]
    if len(prev_winner_now)==0: continue
    ws_drop_off = prev_winner_now['WS'].values[0]-prev_winner['WS'].values[0]
    ps_drop_off = prev_winner_now['POINTS_SHARE'].values[0]-prev_winner['POINTS_SHARE'].values[0]
    # large ws drop off indicates that someone deserved to have a large ps drop off
    chart_df = chart_df.append({l1: ws_drop_off, l2: ps_drop_off, 'Player': prev_winner_name+', '+str(y)}, ignore_index=True)
{% endhighlight %}

**In [4]:**

{% highlight python %}
g = sns.FacetGrid(chart_df, height=15)
order=1
mvp_chart_df = chart_df
x_data = chart_df[l1].values
y_data = chart_df[l2].values
mvp_z = np.polyfit(x_data, y_data, order)
print(mvp_z)
g = g.map(sns.regplot, x=l1, y=l2, data=chart_df, ci=None, order=order)
for row in chart_df.iterrows():
    g.axes[0,0].text(row[1][l1]+0.01, row[1][l2]+0.01, row[1]['Player'], horizontalalignment='center')
    
g.axes[0,0].text(0.2, 0.10, 'Won More and Received More Votes', fontsize=20)
g.axes[0,0].text(-7, 0.10, 'Won Less and Received More Votes', fontsize=20)
g.axes[0,0].text(0.2, -0.4, 'Won More and Received Less Votes', fontsize=20)
g.axes[0,0].text(-7, -0.4, 'Won Less and Received Less Votes', fontsize=20)
g.axes[0, 0].plot([-8, 6], [0, 0], linewidth=2)
g.axes[0, 0].plot([0, 0], [0.1, -1], linewidth=2)
g.axes[0,0].set_xlabel(l1)
g.axes[0,0].set_ylabel(l2)
plt.show()
{% endhighlight %}

    [ 0.08050906 -0.24492895]


 
![png](https://i.imgur.com/vyAU7h5.png) 

 
### DPOY Award

Let's perform the same analysis for the DPOY award. 

**In [5]:**

{% highlight python %}
df = pd.read_csv('dpoy_data.csv')
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
      <th>RANK</th>
      <th>PLAYER</th>
      <th>AGE</th>
      <th>TEAM</th>
      <th>FIRST_PLACE_VOTES</th>
      <th>TOTAL_POINTS_WON</th>
      <th>TOTAL_POINTS_POSSIBLE</th>
      <th>POINTS_SHARE</th>
      <th>G</th>
      <th>MP</th>
      <th>...</th>
      <th>AST</th>
      <th>STL</th>
      <th>BLK</th>
      <th>FG%</th>
      <th>3P%</th>
      <th>FT%</th>
      <th>WS</th>
      <th>WS/48</th>
      <th>SEASON</th>
      <th>DWS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Michael Jordan</td>
      <td>24</td>
      <td>CHI</td>
      <td>37.0</td>
      <td>37.0</td>
      <td>80</td>
      <td>0.463</td>
      <td>82</td>
      <td>40.4</td>
      <td>...</td>
      <td>5.9</td>
      <td>3.2</td>
      <td>1.6</td>
      <td>0.535</td>
      <td>0.132</td>
      <td>0.841</td>
      <td>21.2</td>
      <td>0.308</td>
      <td>1988</td>
      <td>6.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Mark Eaton</td>
      <td>31</td>
      <td>UTA</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>80</td>
      <td>0.113</td>
      <td>82</td>
      <td>33.3</td>
      <td>...</td>
      <td>0.7</td>
      <td>0.5</td>
      <td>3.7</td>
      <td>0.418</td>
      <td>NaN</td>
      <td>0.623</td>
      <td>4.1</td>
      <td>0.072</td>
      <td>1988</td>
      <td>5.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Hakeem Olajuwon</td>
      <td>25</td>
      <td>HOU</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>80</td>
      <td>0.088</td>
      <td>79</td>
      <td>35.8</td>
      <td>...</td>
      <td>2.1</td>
      <td>2.1</td>
      <td>2.7</td>
      <td>0.514</td>
      <td>0.000</td>
      <td>0.695</td>
      <td>10.7</td>
      <td>0.182</td>
      <td>1988</td>
      <td>6.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Alvin Robertson</td>
      <td>25</td>
      <td>SAS</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>80</td>
      <td>0.075</td>
      <td>82</td>
      <td>36.3</td>
      <td>...</td>
      <td>6.8</td>
      <td>3.0</td>
      <td>0.8</td>
      <td>0.465</td>
      <td>0.284</td>
      <td>0.748</td>
      <td>5.8</td>
      <td>0.094</td>
      <td>1988</td>
      <td>2.8</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5T</td>
      <td>Michael Cooper</td>
      <td>31</td>
      <td>LAL</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>80</td>
      <td>0.050</td>
      <td>61</td>
      <td>29.4</td>
      <td>...</td>
      <td>4.7</td>
      <td>1.1</td>
      <td>0.4</td>
      <td>0.392</td>
      <td>0.320</td>
      <td>0.858</td>
      <td>3.1</td>
      <td>0.082</td>
      <td>1988</td>
      <td>1.8</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



**In [6]:**

{% highlight python %}
l1 = 'DWS Year After DPOY - DWS DPOY Year'
l2 = 'PS Year After DPOY - PS DPOY Year'

chart_df = pd.DataFrame(columns=[l1, l2, 'Player'])
for y in df['SEASON'].unique():
    if y==1988: continue
    prev_y = y-1
    prev_y_df = df[df['SEASON']==prev_y]
    prev_winner = prev_y_df[prev_y_df['RANK']=='1']
    if len(prev_winner)==0: continue
    y_df = df[df['SEASON']==y]
    prev_winner_name = prev_winner['PLAYER'].values[0]
    prev_winner_now = y_df[y_df['PLAYER']==prev_winner_name]
    if len(prev_winner_now)==0: continue
    ws_drop_off = prev_winner_now['DWS'].values[0]-prev_winner['DWS'].values[0]
    ps_drop_off = prev_winner_now['POINTS_SHARE'].values[0]-prev_winner['POINTS_SHARE'].values[0]
    # large ws drop off indicates that someone deserved to have a large ps drop off
    chart_df = chart_df.append({l1: ws_drop_off, l2: ps_drop_off, 'Player': prev_winner_name+', '+str(y)}, ignore_index=True)
{% endhighlight %}

**In [7]:**

{% highlight python %}
g = sns.FacetGrid(chart_df, height=15)
order=1
dpoy_chart_df = chart_df
x_data = chart_df[l1].values
y_data = chart_df[l2].values
dpoy_z = np.polyfit(x_data, y_data, order)
print(dpoy_z)
g = g.map(sns.regplot, x=l1, y=l2, data=chart_df, ci=None, order=order)
for row in chart_df.iterrows():
    g.axes[0,0].text(row[1][l1]+0.01, row[1][l2]+0.01, row[1]['Player'], horizontalalignment='center')
    
g.axes[0,0].text(0.2, 0.10, 'Played Better Defense and Received More Votes', fontsize=20)
g.axes[0,0].text(-4, 0.10, 'Played Worse Defense and Received More Votes', fontsize=20)
g.axes[0,0].text(0.2, -0.4, 'Played Better Defense and Received Less Votes', fontsize=20)
g.axes[0,0].text(-4, -0.4, 'Played Worse Defense and Received Less Votes', fontsize=20)
g.axes[0, 0].plot([-4, 2], [0, 0], linewidth=2)
g.axes[0, 0].plot([0, 0], [0.3, -1], linewidth=2)
g.axes[0,0].set_xlabel(l1)
g.axes[0,0].set_ylabel(l2)
plt.show()
{% endhighlight %}

    [ 0.09736021 -0.26654382]


 
![png](https://i.imgur.com/DddmrEt.png) 

 
### MIP Award 

**In [8]:**

{% highlight python %}
df = pd.read_csv('mip_data.csv')
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
      <th>RANK</th>
      <th>PLAYER</th>
      <th>AGE</th>
      <th>TEAM</th>
      <th>FIRST_PLACE_VOTES</th>
      <th>TOTAL_POINTS_WON</th>
      <th>TOTAL_POINTS_POSSIBLE</th>
      <th>POINTS_SHARE</th>
      <th>G</th>
      <th>MP</th>
      <th>...</th>
      <th>AST</th>
      <th>STL</th>
      <th>BLK</th>
      <th>FG%</th>
      <th>3P%</th>
      <th>FT%</th>
      <th>WS</th>
      <th>WS/48</th>
      <th>SEASON</th>
      <th>CURR_PER-PREV_PER</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Kevin Duckworth</td>
      <td>23</td>
      <td>POR</td>
      <td>33.0</td>
      <td>33.0</td>
      <td>80</td>
      <td>0.413</td>
      <td>78</td>
      <td>28.5</td>
      <td>...</td>
      <td>0.8</td>
      <td>0.4</td>
      <td>0.4</td>
      <td>0.496</td>
      <td>NaN</td>
      <td>0.770</td>
      <td>5.2</td>
      <td>0.112</td>
      <td>1988</td>
      <td>5.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>John Stockton</td>
      <td>25</td>
      <td>UTA</td>
      <td>15.0</td>
      <td>15.0</td>
      <td>80</td>
      <td>0.188</td>
      <td>82</td>
      <td>34.7</td>
      <td>...</td>
      <td>13.8</td>
      <td>3.0</td>
      <td>0.2</td>
      <td>0.574</td>
      <td>0.358</td>
      <td>0.840</td>
      <td>14.1</td>
      <td>0.238</td>
      <td>1988</td>
      <td>4.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Mark Price</td>
      <td>23</td>
      <td>CLE</td>
      <td>11.0</td>
      <td>11.0</td>
      <td>80</td>
      <td>0.138</td>
      <td>80</td>
      <td>32.8</td>
      <td>...</td>
      <td>6.0</td>
      <td>1.2</td>
      <td>0.2</td>
      <td>0.506</td>
      <td>0.486</td>
      <td>0.877</td>
      <td>7.8</td>
      <td>0.143</td>
      <td>1988</td>
      <td>6.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Michael Adams</td>
      <td>25</td>
      <td>DEN</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>80</td>
      <td>0.088</td>
      <td>82</td>
      <td>33.9</td>
      <td>...</td>
      <td>6.1</td>
      <td>2.0</td>
      <td>0.2</td>
      <td>0.449</td>
      <td>0.367</td>
      <td>0.834</td>
      <td>7.5</td>
      <td>0.130</td>
      <td>1988</td>
      <td>1.3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Jerome Kersey</td>
      <td>25</td>
      <td>POR</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>80</td>
      <td>0.063</td>
      <td>79</td>
      <td>36.6</td>
      <td>...</td>
      <td>3.1</td>
      <td>1.6</td>
      <td>0.8</td>
      <td>0.499</td>
      <td>0.200</td>
      <td>0.735</td>
      <td>8.6</td>
      <td>0.144</td>
      <td>1988</td>
      <td>0.4</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



**In [9]:**

{% highlight python %}
l1 = 'PER Improvement Year After MIP - PER Improvement MIP Year'
l2 = 'PS Year After MIP - PS MIP Year'

chart_df = pd.DataFrame(columns=[l1, l2, 'Player'])
for y in df['SEASON'].unique():
    if y==1988: continue
    prev_y = y-1
    prev_y_df = df[df['SEASON']==prev_y]
    prev_winner = prev_y_df[prev_y_df['RANK']=='1']
    if len(prev_winner)==0: continue
    y_df = df[df['SEASON']==y]
    prev_winner_name = prev_winner['PLAYER'].values[0]
    prev_winner_now = y_df[y_df['PLAYER']==prev_winner_name]
    if len(prev_winner_now)==0: continue
    ws_drop_off = prev_winner_now['CURR_PER-PREV_PER'].values[0]-prev_winner['CURR_PER-PREV_PER'].values[0]
    ps_drop_off = prev_winner_now['POINTS_SHARE'].values[0]-prev_winner['POINTS_SHARE'].values[0]
    # large ws drop off indicates that someone deserved to have a large ps drop off
    chart_df = chart_df.append({l1: ws_drop_off, l2: ps_drop_off, 'Player': prev_winner_name+', '+str(y)}, ignore_index=True)
{% endhighlight %}

**In [10]:**

{% highlight python %}
g = sns.FacetGrid(chart_df, height=15)
order=1
mip_chart_df = chart_df
x_data = chart_df[l1].values
y_data = chart_df[l2].values
mip_z = np.polyfit(x_data, y_data, order)
print(mip_z)
g = g.map(sns.regplot, x=l1, y=l2, data=chart_df, ci=None, order=order)
for row in chart_df.iterrows():
    g.axes[0,0].text(row[1][l1]+0.01, row[1][l2]+0.01, row[1]['Player'], horizontalalignment='center')
    
g.axes[0,0].text(-6, 0.10, 'Improved More and Received More Votes', fontsize=20)
g.axes[0,0].text(-0.7, 0.10, 'Improved Less and Received More Votes', fontsize=20)
g.axes[0,0].text(-6, -0.4, 'Improved More and Received Less Votes', fontsize=20)
g.axes[0,0].text(-0.7, -0.4, 'Improved Less and Received Less Votes', fontsize=20)
g.axes[0, 0].plot([-6, 3], [0, 0], linewidth=2)
g.axes[0, 0].plot([0, 0], [0.2, -1], linewidth=2)
g.axes[0,0].set_xlabel(l1)
g.axes[0,0].set_ylabel(l2)
plt.show()
{% endhighlight %}

    [ 0.0255688  -0.51181455]


 
![png](https://i.imgur.com/GXPXnxu.png) 

 
Evidently, there is insufficient data to say anything significant about the MIP
award. 
 
### SMOY Award 

**In [11]:**

{% highlight python %}
df = pd.read_csv('smoy_data.csv')
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
      <th>RANK</th>
      <th>PLAYER</th>
      <th>AGE</th>
      <th>TEAM</th>
      <th>FIRST_PLACE_VOTES</th>
      <th>TOTAL_POINTS_WON</th>
      <th>TOTAL_POINTS_POSSIBLE</th>
      <th>POINTS_SHARE</th>
      <th>G</th>
      <th>MP</th>
      <th>...</th>
      <th>AST</th>
      <th>STL</th>
      <th>BLK</th>
      <th>FG%</th>
      <th>3P%</th>
      <th>FT%</th>
      <th>WS</th>
      <th>WS/48</th>
      <th>SEASON</th>
      <th>PER</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Roy Tarpley</td>
      <td>23</td>
      <td>DAL</td>
      <td>67.0</td>
      <td>67.0</td>
      <td>80</td>
      <td>0.838</td>
      <td>81</td>
      <td>28.5</td>
      <td>...</td>
      <td>1.1</td>
      <td>1.3</td>
      <td>1.1</td>
      <td>0.500</td>
      <td>0.000</td>
      <td>0.740</td>
      <td>7.8</td>
      <td>0.163</td>
      <td>1988</td>
      <td>19.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Thurl Bailey</td>
      <td>26</td>
      <td>UTA</td>
      <td>13.0</td>
      <td>13.0</td>
      <td>80</td>
      <td>0.163</td>
      <td>82</td>
      <td>34.2</td>
      <td>...</td>
      <td>1.9</td>
      <td>0.6</td>
      <td>1.5</td>
      <td>0.492</td>
      <td>0.333</td>
      <td>0.826</td>
      <td>6.8</td>
      <td>0.116</td>
      <td>1988</td>
      <td>16.3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>Eddie Johnson</td>
      <td>29</td>
      <td>PHO</td>
      <td>33.0</td>
      <td>33.0</td>
      <td>85</td>
      <td>0.388</td>
      <td>70</td>
      <td>29.2</td>
      <td>...</td>
      <td>2.3</td>
      <td>0.7</td>
      <td>0.1</td>
      <td>0.497</td>
      <td>0.413</td>
      <td>0.868</td>
      <td>6.5</td>
      <td>0.152</td>
      <td>1989</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>Thurl Bailey</td>
      <td>27</td>
      <td>UTA</td>
      <td>26.0</td>
      <td>26.0</td>
      <td>85</td>
      <td>0.306</td>
      <td>82</td>
      <td>33.9</td>
      <td>...</td>
      <td>1.7</td>
      <td>0.6</td>
      <td>1.1</td>
      <td>0.483</td>
      <td>0.400</td>
      <td>0.825</td>
      <td>6.5</td>
      <td>0.113</td>
      <td>1989</td>
      <td>15.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>Dennis Rodman</td>
      <td>27</td>
      <td>DET</td>
      <td>17.0</td>
      <td>17.0</td>
      <td>85</td>
      <td>0.200</td>
      <td>82</td>
      <td>26.9</td>
      <td>...</td>
      <td>1.2</td>
      <td>0.7</td>
      <td>0.9</td>
      <td>0.595</td>
      <td>0.231</td>
      <td>0.626</td>
      <td>8.1</td>
      <td>0.175</td>
      <td>1989</td>
      <td>16.3</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



**In [12]:**

{% highlight python %}
l1 = 'PER Year After SMOY - PER SMOY Year'
l2 = 'PS Year After SMOY - PS SMOY Year'

chart_df = pd.DataFrame(columns=[l1, l2, 'Player'])
for y in df['SEASON'].unique():
    if y==1988: continue
    prev_y = y-1
    prev_y_df = df[df['SEASON']==prev_y]
    prev_winner = prev_y_df[prev_y_df['RANK']=='1']
    if len(prev_winner)==0: continue
    y_df = df[df['SEASON']==y]
    prev_winner_name = prev_winner['PLAYER'].values[0]
    prev_winner_now = y_df[y_df['PLAYER']==prev_winner_name]
    if len(prev_winner_now)==0: continue
    ws_drop_off = prev_winner_now['PER'].values[0]-prev_winner['PER'].values[0]
    ps_drop_off = prev_winner_now['POINTS_SHARE'].values[0]-prev_winner['POINTS_SHARE'].values[0]
    # large ws drop off indicates that someone deserved to have a large ps drop off
    chart_df = chart_df.append({l1: ws_drop_off, l2: ps_drop_off, 'Player': prev_winner_name+', '+str(y)}, ignore_index=True)
{% endhighlight %}

**In [13]:**

{% highlight python %}
chart_df.dropna(inplace=True)
g = sns.FacetGrid(chart_df, height=15)
order=1
smoy_chart_df = chart_df
x_data = chart_df[l1].values
y_data = chart_df[l2].values
smoy_z = np.polyfit(x_data, y_data, order)
print(smoy_z)
g = g.map(sns.regplot, x=l1, y=l2, data=chart_df, ci=None, order=order)
for row in chart_df.iterrows():
    g.axes[0,0].text(row[1][l1]+0.01, row[1][l2]+0.01, row[1]['Player'], horizontalalignment='center')
    
g.axes[0,0].text(0.2, 0.10, 'Improved and Received More Votes', fontsize=20)
g.axes[0,0].text(-6, 0.10, 'Regressed and Received More Votes', fontsize=20)
g.axes[0,0].text(0.2, -0.4, 'Improved and Received Less Votes', fontsize=20)
g.axes[0,0].text(-6, -0.4, 'Regressed and Received Less Votes', fontsize=20)
g.axes[0, 0].plot([-6, 3], [0, 0], linewidth=2)
g.axes[0, 0].plot([0, 0], [0.2, -1], linewidth=2)
g.axes[0,0].set_xlabel(l1)
g.axes[0,0].set_ylabel(l2)
plt.show()
{% endhighlight %}

    [ 0.08159693 -0.33335873]


 
![png](https://i.imgur.com/Auv81vb.png) 

 
### COY Award 

**In [14]:**

{% highlight python %}
df = pd.read_csv('coy_data.csv')
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
      <th>COACH</th>
      <th>POINTS</th>
      <th>SEASON</th>
      <th>TEAM</th>
      <th>W-PW</th>
      <th>POINTS_SHARE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mike Budenholzer</td>
      <td>432</td>
      <td>2019</td>
      <td>MIL</td>
      <td>-1.0</td>
      <td>0.480000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Doc Rivers</td>
      <td>200</td>
      <td>2019</td>
      <td>LAC</td>
      <td>5.0</td>
      <td>0.222222</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mike Malone</td>
      <td>154</td>
      <td>2019</td>
      <td>DEN</td>
      <td>3.0</td>
      <td>0.171111</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Nate McMillan</td>
      <td>62</td>
      <td>2019</td>
      <td>IND</td>
      <td>-2.0</td>
      <td>0.068889</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Kenny Atkinson</td>
      <td>36</td>
      <td>2019</td>
      <td>BRK</td>
      <td>1.0</td>
      <td>0.040000</td>
    </tr>
  </tbody>
</table>
</div>



**In [15]:**

{% highlight python %}
l1 = 'Win Differential After COY - Win Differential COY Year'
l2 = 'PS Year After COY - PS COY Year'

chart_df = pd.DataFrame(columns=[l1, l2, 'Coach'])
for y in df['SEASON'].unique():
    if y==1988: continue
    prev_y = y-1
    prev_y_df = df[df['SEASON']==prev_y]
    prev_winner = prev_y_df[:1]
    if len(prev_winner)==0: continue
    y_df = df[df['SEASON']==y]
    prev_winner_name = prev_winner['COACH'].values[0]
    prev_winner_now = y_df[y_df['COACH']==prev_winner_name]
    if len(prev_winner_now)==0: continue
    ws_drop_off = prev_winner_now['W-PW'].values[0]-prev_winner['W-PW'].values[0]
    ps_drop_off = prev_winner_now['POINTS_SHARE'].values[0]-prev_winner['POINTS_SHARE'].values[0]
    # large ws drop off indicates that someone deserved to have a large ps drop off
    chart_df = chart_df.append({l1: ws_drop_off, l2: ps_drop_off, 'Coach': prev_winner_name+', '+str(y)}, ignore_index=True)
{% endhighlight %}

**In [16]:**

{% highlight python %}
g = sns.FacetGrid(chart_df, height=15)
order=1
x_data = chart_df[l1].values
y_data = chart_df[l2].values
coy_z = np.polyfit(x_data, y_data, order)
print(coy_z)
g = g.map(sns.regplot, x=l1, y=l2, data=chart_df, ci=None, order=order)
for row in chart_df.iterrows():
    g.axes[0,0].text(row[1][l1]+0.01, row[1][l2]+0.01, row[1]['Coach'], horizontalalignment='center')
    
g.axes[0,0].text(0.2, 0.10, 'Team Improved and Received More Votes', fontsize=20)
g.axes[0,0].text(-8, 0.10, 'Team Regressed and Received More Votes', fontsize=20)
g.axes[0,0].text(0.2, -0.2, 'Team Improved and Received Less Votes', fontsize=20)
g.axes[0,0].text(-8, -0.2, 'Team Regressed and Received Less Votes', fontsize=20)
g.axes[0, 0].plot([-8, 6], [0, 0], linewidth=2)
g.axes[0, 0].plot([0, 0], [0.2, -0.6], linewidth=2)
g.axes[0,0].set_xlabel(l1)
g.axes[0,0].set_ylabel(l2)
plt.show()
{% endhighlight %}

    [-0.00767806 -0.37150997]


 
![png](https://i.imgur.com/bEnvGoe.png) 

 
Two data points consisting of Steve Kerr and Mike D'Antoni is insufficient, so
we can't really say much about COY. 
 
## Summary 
 
Let's take a look at all the best fit lines along with the corresponding
x-intercepts for the awards we *can* say something about (MVP, DPOY, SMOY). 

**In [39]:**

{% highlight python %}
def shot_line_stats(label, z):
    print(label.upper())
    print('Best fit line:')
    print(f'y={z[0]}*x+({z[1]})')
    x_intercept = -z[1]/z[0]
    print(f'x-intercept: {x_intercept}' )
    print('\n')
{% endhighlight %}

**In [40]:**

{% highlight python %}
shot_line_stats('mvp', mvp_z)
shot_line_stats('dopy', dpoy_z)
shot_line_stats('smoy', smoy_z)
{% endhighlight %}

    MVP
    Best fit line:
    y=0.08050905774386342*x+(-0.24492895462768918)
    x-intercept: 3.042253399697231
    
    
    DOPY
    Best fit line:
    y=0.09736020500961524*x+(-0.26654381850154735)
    x-intercept: 2.7377080653766455
    
    
    SMOY
    Best fit line:
    y=0.08159692715800439*x+(-0.33335872670259853)
    x-intercept: 4.085432360180455
    
    

 
# Conclusions and Takeaways 
 
So what does this mean for each of the awards listed above?

**To repeat as MVP, an individual must increase his win shares by ~3.042.**

**To repeat as DPOY, an individual must increase his defensive win shares by
~2.738.**

**To repeat as SMOY, an individual must increase his PER by ~4.085.** 
 
Now, not all of these statistics are on the same scale. Of course, Win Shares
and Defensive Win Shares are on a similar scale so we can compare them
relatively easily.

PER, on the other hand, is on a separate scale. However, we can easily eyeball
the scale by comparing the relative PER and WS of top recent candidates.

Consider Giannis Antetokounmpo. His WS is 8.1 PER is 32.7

Consider James Harden. His WS is 8.7 PER is 29.4

Generally, the same pattern holds where a player's PER is approximately 3-4x his
WS. 
 
Evidently, WS and DWS is a metric **signficantly** harder to increase than PER.
So increasing a player's PER by ~4 is signficantly easier than increasing a
player's WS or DWS by ~3.

As a result, we can fairly certainly say that the difficulty to repeat in one of
the three awards (i.e. get the same amount of POINTS_SHARE) it can be ordered in
difficulty as follows:

**1) MVP**

**2) DPOY**

**3) SMOY**

With MVP and DPOY being of similar difficulty, but SMOY being significantly
easier.

We can, thus, conclude that voter fatigue affects the MVP award the most,
followed by DPOY and then SMOY. 
