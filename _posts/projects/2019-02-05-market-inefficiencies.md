---
title: "Exploring NBA Betting Market Inefficiencies: Perimeter Defense"
date: 2019-02-05
permalink: /projects/2019/02/05/market-inefficiencies
tags:
    - python
    - notebook
--- 

## Introduction
In this post, I wanted to explore Perimeter Defense as a market inefficiency in
NBA sports betting as of recent years. The idea was inspired by Ethan Sherwood
Strauss' posts in [Strauss vs. The
House](https://theathletic.com/745690/2019/01/02/strauss-vs-the-
house-a-2019-hiatus/) posts and podcasts where he argues that Perimeter Defense
is a market inefficiency.

## Data Aggregation

To test his theory, I aggregated all the Westgate odds from the 2013-2014
regular season to the 2017-2018 regular season and checked when Westgate
**incorrectly** predicted the winner of an individual game.

Next, I gave each team a regular season *Perimeter Defensive Score*. This value
was calculated by looking at the Player Grades data from [bball-
index](https://www.bball-index.com/). For example, the 2013-2014 player grades
can be found [here](https://www.bball-index.com/2013-14-player-grades/) along
with explanation of how values are calculated.

## Quantifying Team Perimeter Defense

A *team's* regular season *Perimeter Defensive Score* was calculated as the
average of all players classified as a *wing* or *guard* on an individual team,
weighted by minutes played.

For example, here are the calculations for the 2017-2018 Golden State Warriors:

&#x200B;

|Player|Minutes Played|Score|
|:-|:-|:-|
|Klay Thompson|3300|0.447|
|Kevin Durant|3132|0.351|
|Stephen Curry|2186|0.76|
|Andre Iguodala|2023|0.718|
|Nick Young|1598|0.5|
|Shaun Livingston|1491|0.419|
|Patrick McCaw|977|0.639|
|Quinn Cook|915|0.382|
|Omri Casspi|740|0.424|
|Chris Boucher|1|0.0|
 
 
Let's perform this computation for all players in the 2017-18 season, for
example. 

**In [2]:**

{% highlight python %}
import csv
d = {}
YEAR = '2017_18'
with open(f'{YEAR}.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        if row['TEAM'] not in d:
            d[row['TEAM']] = []
        if row['ADVANCED_POSITION']=='Wing' or row['ADVANCED_POSITION']=='Guard':
            d[row['TEAM']].append({'minutes': int(row['NUMDATA']), 'grade': float(row['PERIMETER_DEFENSE'])})
            
print(d)
{% endhighlight %}

    {'CLE': [{'minutes': 3948, 'grade': 0.5429999999999999}, {'minutes': 2950, 'grade': 0.445}, {'minutes': 2079, 'grade': 0.524}, {'minutes': 1018, 'grade': 0.6759999999999999}, {'minutes': 734, 'grade': 0.7170000000000001}, {'minutes': 276, 'grade': 0.605}, {'minutes': 174, 'grade': 0.564}, {'minutes': 66, 'grade': 0.542}], 'GSW': [{'minutes': 3300, 'grade': 0.447}, {'minutes': 3132, 'grade': 0.35100000000000003}, {'minutes': 2186, 'grade': 0.76}, {'minutes': 2023, 'grade': 0.718}, {'minutes': 1598, 'grade': 0.5}, {'minutes': 1491, 'grade': 0.419}, {'minutes': 977, 'grade': 0.639}, {'minutes': 915, 'grade': 0.382}, {'minutes': 740, 'grade': 0.424}, {'minutes': 1, 'grade': 0.0}], 'NOP': [{'minutes': 3275, 'grade': 0.835}, {'minutes': 2870, 'grade': 0.439}, {'minutes': 2106, 'grade': 0.33899999999999997}, {'minutes': 2007, 'grade': 0.7559999999999999}, {'minutes': 1645, 'grade': 0.433}, {'minutes': 301, 'grade': 0.33799999999999997}, {'minutes': 273, 'grade': 0.583}, {'minutes': 68, 'grade': 0.413}, {'minutes': 38, 'grade': 0.444}, {'minutes': 35, 'grade': 0.0}], 'MIL': [{'minutes': 3257, 'grade': 0.622}, {'minutes': 2187, 'grade': 0.491}, {'minutes': 1622, 'grade': 0.441}, {'minutes': 910, 'grade': 0.597}, {'minutes': 858, 'grade': 0.715}, {'minutes': 790, 'grade': 0.37}, {'minutes': 789, 'grade': 0.7759999999999999}, {'minutes': 210, 'grade': 0.519}, {'minutes': 21, 'grade': 0.611}], 'WAS': [{'minutes': 3193, 'grade': 0.69}, {'minutes': 2590, 'grade': 0.685}, {'minutes': 2379, 'grade': 0.611}, {'minutes': 1703, 'grade': 0.604}, {'minutes': 1644, 'grade': 0.728}, {'minutes': 1119, 'grade': 0.501}, {'minutes': 844, 'grade': 0.7829999999999999}, {'minutes': 90, 'grade': 0.125}], 'HOU': [{'minutes': 3172, 'grade': 0.705}, {'minutes': 2851, 'grade': 0.75}, {'minutes': 2703, 'grade': 0.51}, {'minutes': 2364, 'grade': 0.8079999999999999}, {'minutes': 1820, 'grade': 0.203}, {'minutes': 1713, 'grade': 0.741}, {'minutes': 1201, 'grade': 0.358}, {'minutes': 115, 'grade': 0.467}, {'minutes': 31, 'grade': 0.363}, {'minutes': 25, 'grade': 0.462}], 'OKC': [{'minutes': 3149, 'grade': 0.828}, {'minutes': 3142, 'grade': 0.7}, {'minutes': 1780, 'grade': 0.308}, {'minutes': 1444, 'grade': 0.711}, {'minutes': 1244, 'grade': 0.595}, {'minutes': 1037, 'grade': 0.857}, {'minutes': 769, 'grade': 0.642}, {'minutes': 59, 'grade': 0.321}, {'minutes': 28, 'grade': 0.508}, {'minutes': 3, 'grade': 0.0}], 'MIN': [{'minutes': 3143, 'grade': 0.491}, {'minutes': 2464, 'grade': 0.622}, {'minutes': 2334, 'grade': 0.8740000000000001}, {'minutes': 1776, 'grade': 0.402}, {'minutes': 1522, 'grade': 0.8959999999999999}, {'minutes': 1418, 'grade': 0.5429999999999999}, {'minutes': 227, 'grade': 0.518}, {'minutes': 192, 'grade': 0.537}], 'BOS': [{'minutes': 3121, 'grade': 0.6759999999999999}, {'minutes': 2764, 'grade': 0.645}, {'minutes': 2735, 'grade': 0.6940000000000001}, {'minutes': 2063, 'grade': 0.8490000000000001}, {'minutes': 2009, 'grade': 0.371}, {'minutes': 1931, 'grade': 0.614}, {'minutes': 1380, 'grade': 0.39399999999999996}, {'minutes': 929, 'grade': 0.62}, {'minutes': 555, 'grade': 0.55}, {'minutes': 115, 'grade': 0.598}, {'minutes': 107, 'grade': 0.613}, {'minutes': 5, 'grade': 0.0}], 'PHI': [{'minutes': 3101, 'grade': 0.7829999999999999}, {'minutes': 2813, 'grade': 0.818}, {'minutes': 2458, 'grade': 0.375}, {'minutes': 1861, 'grade': 0.733}, {'minutes': 927, 'grade': 0.384}, {'minutes': 807, 'grade': 0.41100000000000003}, {'minutes': 552, 'grade': 0.5920000000000001}, {'minutes': 276, 'grade': 0.785}, {'minutes': 82, 'grade': 0.313}, {'minutes': 61, 'grade': 0.147}, {'minutes': 18, 'grade': 0.0}, {'minutes': 6, 'grade': 0.18}], 'POR': [{'minutes': 3078, 'grade': 0.515}, {'minutes': 2832, 'grade': 0.61}, {'minutes': 2203, 'grade': 0.574}, {'minutes': 2121, 'grade': 0.317}, {'minutes': 1570, 'grade': 0.679}, {'minutes': 1547, 'grade': 0.42700000000000005}, {'minutes': 1317, 'grade': 0.568}, {'minutes': 168, 'grade': 0.534}, {'minutes': 99, 'grade': 0.5329999999999999}], 'TOR': [{'minutes': 3065, 'grade': 0.541}, {'minutes': 2871, 'grade': 0.601}, {'minutes': 1719, 'grade': 0.542}, {'minutes': 1648, 'grade': 0.818}, {'minutes': 1634, 'grade': 0.81}, {'minutes': 1564, 'grade': 0.5589999999999999}, {'minutes': 1102, 'grade': 0.672}, {'minutes': 168, 'grade': 0.705}, {'minutes': 126, 'grade': 0.34700000000000003}, {'minutes': 53, 'grade': 0.267}], 'UTA': [{'minutes': 3049, 'grade': 0.778}, {'minutes': 2960, 'grade': 0.6459999999999999}, {'minutes': 2435, 'grade': 0.8290000000000001}, {'minutes': 1409, 'grade': 0.45899999999999996}, {'minutes': 1179, 'grade': 0.585}, {'minutes': 806, 'grade': 0.684}, {'minutes': 570, 'grade': 0.527}, {'minutes': 349, 'grade': 0.519}, {'minutes': 32, 'grade': 0.5770000000000001}, {'minutes': 19, 'grade': 0.272}, {'minutes': 1, 'grade': 0.0}], 'IND': [{'minutes': 2813, 'grade': 0.9490000000000001}, {'minutes': 2702, 'grade': 0.53}, {'minutes': 2353, 'grade': 0.631}, {'minutes': 2232, 'grade': 0.852}, {'minutes': 1999, 'grade': 0.45899999999999996}, {'minutes': 561, 'grade': 0.563}, {'minutes': 463, 'grade': 0.23600000000000002}, {'minutes': 344, 'grade': 0.6509999999999999}, {'minutes': 152, 'grade': 0.295}], 'MIA': [{'minutes': 2819, 'grade': 0.76}, {'minutes': 2534, 'grade': 0.545}, {'minutes': 2142, 'grade': 0.451}, {'minutes': 2133, 'grade': 0.46399999999999997}, {'minutes': 1805, 'grade': 0.653}, {'minutes': 918, 'grade': 0.43}, {'minutes': 315, 'grade': 0.542}, {'minutes': 147, 'grade': 0.625}, {'minutes': 80, 'grade': 0.201}, {'minutes': 11, 'grade': 0.336}], 'CHA': [{'minutes': 2736, 'grade': 0.594}, {'minutes': 2006, 'grade': 0.5}, {'minutes': 1981, 'grade': 0.47700000000000004}, {'minutes': 1967, 'grade': 0.71}, {'minutes': 1850, 'grade': 0.39799999999999996}, {'minutes': 1050, 'grade': 0.499}, {'minutes': 854, 'grade': 0.349}, {'minutes': 835, 'grade': 0.7559999999999999}, {'minutes': 713, 'grade': 0.44799999999999995}, {'minutes': 175, 'grade': 0.522}, {'minutes': 28, 'grade': 0.19399999999999998}], 'SAS': [{'minutes': 2272, 'grade': 0.408}, {'minutes': 2051, 'grade': 0.787}, {'minutes': 1894, 'grade': 0.628}, {'minutes': 1839, 'grade': 0.731}, {'minutes': 1571, 'grade': 0.439}, {'minutes': 1406, 'grade': 0.635}, {'minutes': 1391, 'grade': 0.6990000000000001}, {'minutes': 1168, 'grade': 0.48200000000000004}, {'minutes': 1138, 'grade': 0.408}, {'minutes': 579, 'grade': 0.762}, {'minutes': 210, 'grade': 0.799}, {'minutes': 157, 'grade': 0.585}, {'minutes': 95, 'grade': 0.34700000000000003}], 'DEN': [{'minutes': 2683, 'grade': 0.475}, {'minutes': 2565, 'grade': 0.5}, {'minutes': 2346, 'grade': 0.34299999999999997}, {'minutes': 2304, 'grade': 0.883}, {'minutes': 629, 'grade': 0.379}, {'minutes': 583, 'grade': 0.39}, {'minutes': 277, 'grade': 0.47100000000000003}, {'minutes': 163, 'grade': 0.34700000000000003}, {'minutes': 25, 'grade': 0.648}], 'TOT': [{'minutes': 2668, 'grade': 0.425}, {'minutes': 2547, 'grade': 0.892}, {'minutes': 2413, 'grade': 0.602}, {'minutes': 2369, 'grade': 0.654}, {'minutes': 2220, 'grade': 0.526}, {'minutes': 2174, 'grade': 0.581}, {'minutes': 1876, 'grade': 0.433}, {'minutes': 1808, 'grade': 0.69}, {'minutes': 1768, 'grade': 0.42700000000000005}, {'minutes': 1663, 'grade': 0.745}, {'minutes': 1604, 'grade': 0.574}, {'minutes': 1562, 'grade': 0.423}, {'minutes': 1433, 'grade': 0.721}, {'minutes': 1359, 'grade': 0.772}, {'minutes': 1340, 'grade': 0.7020000000000001}, {'minutes': 1259, 'grade': 0.27899999999999997}, {'minutes': 1245, 'grade': 0.627}, {'minutes': 1013, 'grade': 0.47100000000000003}, {'minutes': 862, 'grade': 0.35200000000000004}, {'minutes': 738, 'grade': 0.6829999999999999}, {'minutes': 718, 'grade': 0.203}, {'minutes': 687, 'grade': 0.6920000000000001}, {'minutes': 640, 'grade': 0.424}, {'minutes': 539, 'grade': 0.268}, {'minutes': 523, 'grade': 0.303}, {'minutes': 447, 'grade': 0.45899999999999996}, {'minutes': 422, 'grade': 0.653}, {'minutes': 392, 'grade': 0.65}, {'minutes': 324, 'grade': 0.562}, {'minutes': 307, 'grade': 0.728}, {'minutes': 245, 'grade': 0.309}, {'minutes': 237, 'grade': 0.755}, {'minutes': 221, 'grade': 0.598}, {'minutes': 212, 'grade': 0.317}, {'minutes': 181, 'grade': 0.5329999999999999}, {'minutes': 140, 'grade': 0.523}, {'minutes': 123, 'grade': 0.794}, {'minutes': 107, 'grade': 0.428}, {'minutes': 98, 'grade': 0.6990000000000001}, {'minutes': 80, 'grade': 0.706}], 'DAL': [{'minutes': 2634, 'grade': 0.289}, {'minutes': 2282, 'grade': 0.483}, {'minutes': 2131, 'grade': 0.5489999999999999}, {'minutes': 2049, 'grade': 0.5670000000000001}, {'minutes': 1603, 'grade': 0.344}, {'minutes': 1206, 'grade': 0.396}, {'minutes': 480, 'grade': 0.69}, {'minutes': 448, 'grade': 0.428}, {'minutes': 233, 'grade': 0.6459999999999999}, {'minutes': 64, 'grade': 0.652}, {'minutes': 27, 'grade': 0.18600000000000003}, {'minutes': 8, 'grade': 0.35200000000000004}], 'DET': [{'minutes': 2043, 'grade': 0.6679999999999999}, {'minutes': 1894, 'grade': 0.659}, {'minutes': 1732, 'grade': 0.604}, {'minutes': 1463, 'grade': 0.541}, {'minutes': 1201, 'grade': 0.544}, {'minutes': 863, 'grade': 0.805}, {'minutes': 427, 'grade': 0.845}, {'minutes': 136, 'grade': 0.159}, {'minutes': 8, 'grade': 0.22899999999999998}, {'minutes': 7, 'grade': 0.183}], 'LAC': [{'minutes': 2589, 'grade': 0.6829999999999999}, {'minutes': 2057, 'grade': 0.61}, {'minutes': 1486, 'grade': 0.809}, {'minutes': 1156, 'grade': 0.8170000000000001}, {'minutes': 1134, 'grade': 0.33}, {'minutes': 883, 'grade': 0.499}, {'minutes': 851, 'grade': 0.537}, {'minutes': 778, 'grade': 0.82}, {'minutes': 707, 'grade': 0.716}, {'minutes': 671, 'grade': 0.48700000000000004}, {'minutes': 334, 'grade': 0.821}, {'minutes': 274, 'grade': 0.39399999999999996}], 'ATL': [{'minutes': 2464, 'grade': 0.45299999999999996}, {'minutes': 2078, 'grade': 0.544}, {'minutes': 1789, 'grade': 0.7070000000000001}, {'minutes': 1167, 'grade': 0.5870000000000001}, {'minutes': 1014, 'grade': 0.687}, {'minutes': 974, 'grade': 0.337}, {'minutes': 455, 'grade': 0.695}, {'minutes': 404, 'grade': 0.7979999999999999}, {'minutes': 216, 'grade': 0.5710000000000001}, {'minutes': 209, 'grade': 0.373}, {'minutes': 98, 'grade': 0.48100000000000004}, {'minutes': 10, 'grade': 0.306}], 'LAL': [{'minutes': 2458, 'grade': 0.615}, {'minutes': 2401, 'grade': 0.23600000000000002}, {'minutes': 1975, 'grade': 0.496}, {'minutes': 1780, 'grade': 0.8220000000000001}, {'minutes': 1461, 'grade': 0.447}, {'minutes': 683, 'grade': 0.606}, {'minutes': 562, 'grade': 0.748}, {'minutes': 228, 'grade': 0.33}, {'minutes': 45, 'grade': 0.43700000000000006}, {'minutes': 9, 'grade': 0.163}], 'MEM': [{'minutes': 2350, 'grade': 0.583}, {'minutes': 1607, 'grade': 0.758}, {'minutes': 1542, 'grade': 0.381}, {'minutes': 1421, 'grade': 0.8320000000000001}, {'minutes': 1326, 'grade': 0.696}, {'minutes': 1091, 'grade': 0.616}, {'minutes': 692, 'grade': 0.452}, {'minutes': 691, 'grade': 0.39799999999999996}, {'minutes': 643, 'grade': 0.46399999999999997}, {'minutes': 378, 'grade': 0.8440000000000001}, {'minutes': 373, 'grade': 0.713}, {'minutes': 118, 'grade': 0.782}, {'minutes': 7, 'grade': 0.134}], 'NYK': [{'minutes': 2310, 'grade': 0.669}, {'minutes': 1885, 'grade': 0.61}, {'minutes': 1706, 'grade': 0.767}, {'minutes': 1653, 'grade': 0.47600000000000003}, {'minutes': 1548, 'grade': 0.40700000000000003}, {'minutes': 1353, 'grade': 0.442}, {'minutes': 785, 'grade': 0.5539999999999999}, {'minutes': 474, 'grade': 0.537}, {'minutes': 385, 'grade': 0.8270000000000001}, {'minutes': 326, 'grade': 0.257}, {'minutes': 2, 'grade': 0.0}], 'BKN': [{'minutes': 2306, 'grade': 0.4}, {'minutes': 2197, 'grade': 0.429}, {'minutes': 2180, 'grade': 0.414}, {'minutes': 1975, 'grade': 0.27399999999999997}, {'minutes': 1864, 'grade': 0.581}, {'minutes': 1234, 'grade': 0.555}, {'minutes': 180, 'grade': 0.6729999999999999}, {'minutes': 125, 'grade': 0.688}, {'minutes': 120, 'grade': 0.175}, {'minutes': 25, 'grade': 0.428}], 'CHI': [{'minutes': 2265, 'grade': 0.604}, {'minutes': 2095, 'grade': 0.674}, {'minutes': 1686, 'grade': 0.715}, {'minutes': 1646, 'grade': 0.6809999999999999}, {'minutes': 1525, 'grade': 0.8009999999999999}, {'minutes': 824, 'grade': 0.43799999999999994}, {'minutes': 656, 'grade': 0.664}, {'minutes': 582, 'grade': 0.7559999999999999}, {'minutes': 314, 'grade': 0.45299999999999996}, {'minutes': 304, 'grade': 0.825}, {'minutes': 196, 'grade': 0.706}], 'SAC': [{'minutes': 2175, 'grade': 0.59}, {'minutes': 2026, 'grade': 0.5660000000000001}, {'minutes': 2024, 'grade': 0.6940000000000001}, {'minutes': 1615, 'grade': 0.637}, {'minutes': 1506, 'grade': 0.306}, {'minutes': 1026, 'grade': 0.652}, {'minutes': 984, 'grade': 0.632}, {'minutes': 344, 'grade': 0.461}], 'PHX': [{'minutes': 2142, 'grade': 0.617}, {'minutes': 1959, 'grade': 0.5479999999999999}, {'minutes': 1865, 'grade': 0.37200000000000005}, {'minutes': 1658, 'grade': 0.6509999999999999}, {'minutes': 1622, 'grade': 0.314}, {'minutes': 686, 'grade': 0.5489999999999999}, {'minutes': 403, 'grade': 0.41200000000000003}, {'minutes': 384, 'grade': 0.882}, {'minutes': 242, 'grade': 0.6729999999999999}, {'minutes': 86, 'grade': 0.735}], 'ORL': [{'minutes': 2029, 'grade': 0.44}, {'minutes': 1837, 'grade': 0.442}, {'minutes': 1760, 'grade': 0.605}, {'minutes': 1657, 'grade': 0.718}, {'minutes': 1365, 'grade': 0.7909999999999999}, {'minutes': 1020, 'grade': 0.5429999999999999}, {'minutes': 682, 'grade': 0.299}, {'minutes': 600, 'grade': 0.789}, {'minutes': 536, 'grade': 0.6809999999999999}, {'minutes': 290, 'grade': 0.504}, {'minutes': 279, 'grade': 0.373}]}

 
So the *defensive score* calculated for the 2017-18 GSW was:
```
    score = (3300*0.447+3132*0.35100000000000003+2186*0.76+2023*0.718+1598*0.5+1
491*0.419+977*0.639+915*0.382+740*0.424+1*0.0)/16363 = 0.5133305628552222
``` 
 
Let's perform the same computation for *all* teams. 

**In [4]:**

{% highlight python %}
abbr = {}
with open('abbr.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        key = ''.join(row['FULL_NAME'].split(' ')[:-1])
        abbr[key] = row['ABBR']
abbr['LALakers'] = 'LAL'
abbr['LAClippers'] = 'LAC'
abbr['Portland'] = 'POR'
print(abbr)
{% endhighlight %}

    {'Atlanta': 'ATL', 'Brooklyn': 'BKN', 'Boston': 'BOS', 'Charlotte': 'CHA', 'Chicago': 'CHI', 'Cleveland': 'CLE', 'Dallas': 'DAL', 'Denver': 'DEN', 'Detroit': 'DET', 'GoldenState': 'GSW', 'Houston': 'HOU', 'Indiana': 'IND', 'LosAngeles': 'LAL', 'Memphis': 'MEM', 'Miami': 'MIA', 'Milwaukee': 'MIL', 'Minnesota': 'MIN', 'NewOrleans': 'NOP', 'NewYork': 'NYK', 'OklahomaCity': 'OKC', 'Orlando': 'ORL', 'Philadelphia': 'PHI', 'Phoenix': 'PHX', 'PortlandTrail': 'POR', 'Sacramento': 'SAC', 'SanAntonio': 'SAS', 'Toronto': 'TOR', 'Utah': 'UTA', 'Washington': 'WAS', 'LALakers': 'LAL', 'LAClippers': 'LAC', 'Portland': 'POR'}


**In [5]:**

{% highlight python %}
d2 = {}
s = ''
for t in d:
    minutes = [e['minutes'] for e in d[t]]
    grades = [e['grade'] for e in d[t]]
    v = sum([minutes[i]*grades[i]/sum(minutes) for i in range(len(minutes))])
    d2[t] = v

if 'BKN' not in d2:
    d2['BKN'] = d2['BRK'] 
    
if 'PHX' not in d2:
    d2['PHX'] = d2['PHO']

if 'CHA' not in d2:
    d2['CHA'] = d2['CHO']
print(d2)
{% endhighlight %}

    {'CLE': 0.5390168074699865, 'GSW': 0.5133305628552222, 'NOP': 0.5740940719606911, 'MIL': 0.5635191657271702, 'WAS': 0.6554382834390208, 'HOU': 0.6132173179118474, 'OKC': 0.674630264717503, 'MIN': 0.6258844447843378, 'BOS': 0.6228089082081968, 'PHI': 0.6389277889214628, 'POR': 0.5267293605624374, 'TOR': 0.6292524014336919, 'UTA': 0.6788820360683895, 'IND': 0.668143035465159, 'MIA': 0.568429169249845, 'CHA': 0.5336705177879535, 'SAS': 0.5954422674529198, 'DEN': 0.5239761555075593, 'TOT': 0.5705035499457647, 'DAL': 0.4517409798708697, 'DET': 0.632708410067526, 'LAC': 0.6419989164086688, 'ATL': 0.5618165103879389, 'LAL': 0.5261833304602654, 'MEM': 0.6142030394640086, 'NYK': 0.5721603765993403, 'BKN': 0.43546427986236275, 'CHI': 0.6694730009096169, 'SAC': 0.5789433333333334, 'PHX': 0.5236721281795963, 'ORL': 0.5711635006221484}

 
Let's look at the aggregated Westgate odds from the 2013-2014 regular season to
the 2017-2018 regular season and checked when Westgate **incorrectly** predicted
the winner of an individual game. 

**In [7]:**

{% highlight python %}
import datetime
data = {}
with open(f'odds_{YEAR}.csv', 'r', newline='\r') as f:
    reader = csv.DictReader(f)
    i = 0
    r1 = None
    r2 = None
    for row in reader:
        if r1 is None:
            r1 = row
        elif r2 is None:
            r2 = row
        if r1 is not None and r2 is not None:
            t1 = r1['Team']
            t2 = r2['Team']
            ml1 = float(r1['ML'])
            ml2 = float(r2['ML'])
            if ml1<0:
                exp_winner = t1
                exp_loser = t2
            else:
                exp_winner = t2
                exp_loser = t1
            if float(r1['Final'])>float(r2['Final']):
                winner = t1
                loser = t2
            else:
                winner = t2
                loser = t1
            if exp_winner!=winner:
                winner = winner.replace(' ','')
                loser = loser.replace(' ','')
                w_key = abbr[winner]
                l_key = abbr[loser]
                v = d2[w_key] - d2[l_key]
                date_str = r1['\ufeffDate']
                if len(date_str)==4:
                    month = int(date_str[0:2])
                    day = int(date_str[2:])
                    year = int(YEAR[0:4])
                else:
                    month = int(date_str[0:1])
                    day = int(date_str[1:])
                    year = int('20'+YEAR[4:])
                date = datetime.date(year, month, day)
                data[date] = {'winner': abbr[winner], 'loser': abbr[loser], 'winner_val': d2[w_key], 'loser_val': d2[l_key]}
            r1 = None
            r2 = None
print(data)
{% endhighlight %}

    {datetime.date(2017, 10, 17): {'winner': 'HOU', 'loser': 'GSW', 'winner_val': 0.6132173179118474, 'loser_val': 0.5133305628552222}, datetime.date(2017, 10, 18): {'winner': 'ATL', 'loser': 'DAL', 'winner_val': 0.5618165103879389, 'loser_val': 0.4517409798708697}, datetime.date(2017, 10, 20): {'winner': 'LAL', 'loser': 'PHX', 'winner_val': 0.5261833304602654, 'loser_val': 0.5236721281795963}, datetime.date(2017, 10, 21): {'winner': 'UTA', 'loser': 'OKC', 'winner_val': 0.6788820360683895, 'loser_val': 0.674630264717503}, datetime.date(2017, 10, 22): {'winner': 'MIN', 'loser': 'OKC', 'winner_val': 0.6258844447843378, 'loser_val': 0.674630264717503}, datetime.date(2017, 10, 23): {'winner': 'PHX', 'loser': 'SAC', 'winner_val': 0.5236721281795963, 'loser_val': 0.5789433333333334}, datetime.date(2017, 10, 24): {'winner': 'IND', 'loser': 'MIN', 'winner_val': 0.668143035465159, 'loser_val': 0.6258844447843378}, datetime.date(2017, 10, 25): {'winner': 'LAL', 'loser': 'WAS', 'winner_val': 0.5261833304602654, 'loser_val': 0.6554382834390208}, datetime.date(2017, 10, 26): {'winner': 'NOP', 'loser': 'SAC', 'winner_val': 0.5740940719606911, 'loser_val': 0.5789433333333334}, datetime.date(2017, 10, 27): {'winner': 'MIN', 'loser': 'OKC', 'winner_val': 0.6258844447843378, 'loser_val': 0.674630264717503}, datetime.date(2017, 10, 28): {'winner': 'DET', 'loser': 'LAC', 'winner_val': 0.632708410067526, 'loser_val': 0.6419989164086688}, datetime.date(2017, 10, 29): {'winner': 'DET', 'loser': 'GSW', 'winner_val': 0.632708410067526, 'loser_val': 0.5133305628552222}, datetime.date(2017, 10, 30): {'winner': 'TOR', 'loser': 'POR', 'winner_val': 0.6292524014336919, 'loser_val': 0.5267293605624374}, datetime.date(2017, 10, 31): {'winner': 'LAL', 'loser': 'DET', 'winner_val': 0.5261833304602654, 'loser_val': 0.632708410067526}, datetime.date(2017, 11, 1): {'winner': 'MIN', 'loser': 'NOP', 'winner_val': 0.6258844447843378, 'loser_val': 0.5740940719606911}, datetime.date(2017, 11, 3): {'winner': 'BOS', 'loser': 'OKC', 'winner_val': 0.6228089082081968, 'loser_val': 0.674630264717503}, datetime.date(2017, 11, 4): {'winner': 'MEM', 'loser': 'LAC', 'winner_val': 0.6142030394640086, 'loser_val': 0.6419989164086688}, datetime.date(2017, 11, 5): {'winner': 'LAL', 'loser': 'MEM', 'winner_val': 0.5261833304602654, 'loser_val': 0.6142030394640086}, datetime.date(2017, 11, 7): {'winner': 'SAC', 'loser': 'OKC', 'winner_val': 0.5789433333333334, 'loser_val': 0.674630264717503}, datetime.date(2017, 11, 9): {'winner': 'DEN', 'loser': 'OKC', 'winner_val': 0.5239761555075593, 'loser_val': 0.674630264717503}, datetime.date(2017, 11, 10): {'winner': 'BKN', 'loser': 'POR', 'winner_val': 0.43546427986236275, 'loser_val': 0.5267293605624374}, datetime.date(2017, 11, 11): {'winner': 'PHX', 'loser': 'MIN', 'winner_val': 0.5236721281795963, 'loser_val': 0.6258844447843378}, datetime.date(2017, 11, 12): {'winner': 'BOS', 'loser': 'TOR', 'winner_val': 0.6228089082081968, 'loser_val': 0.6292524014336919}, datetime.date(2017, 11, 13): {'winner': 'PHI', 'loser': 'LAC', 'winner_val': 0.6389277889214628, 'loser_val': 0.6419989164086688}, datetime.date(2017, 11, 14): {'winner': 'TOR', 'loser': 'HOU', 'winner_val': 0.6292524014336919, 'loser_val': 0.6132173179118474}, datetime.date(2017, 11, 15): {'winner': 'IND', 'loser': 'MEM', 'winner_val': 0.668143035465159, 'loser_val': 0.6142030394640086}, datetime.date(2017, 11, 16): {'winner': 'BOS', 'loser': 'GSW', 'winner_val': 0.6228089082081968, 'loser_val': 0.5133305628552222}, datetime.date(2017, 11, 17): {'winner': 'PHX', 'loser': 'LAL', 'winner_val': 0.5236721281795963, 'loser_val': 0.5261833304602654}, datetime.date(2017, 11, 18): {'winner': 'DAL', 'loser': 'MIL', 'winner_val': 0.4517409798708697, 'loser_val': 0.5635191657271702}, datetime.date(2017, 11, 19): {'winner': 'LAL', 'loser': 'DEN', 'winner_val': 0.5261833304602654, 'loser_val': 0.5239761555075593}, datetime.date(2017, 11, 20): {'winner': 'NOP', 'loser': 'OKC', 'winner_val': 0.5740940719606911, 'loser_val': 0.674630264717503}, datetime.date(2017, 11, 22): {'winner': 'SAC', 'loser': 'LAL', 'winner_val': 0.5789433333333334, 'loser_val': 0.5261833304602654}, datetime.date(2017, 11, 24): {'winner': 'IND', 'loser': 'TOR', 'winner_val': 0.668143035465159, 'loser_val': 0.6292524014336919}, datetime.date(2017, 11, 25): {'winner': 'UTA', 'loser': 'MIL', 'winner_val': 0.6788820360683895, 'loser_val': 0.5635191657271702}, datetime.date(2017, 11, 26): {'winner': 'BKN', 'loser': 'MEM', 'winner_val': 0.43546427986236275, 'loser_val': 0.6142030394640086}, datetime.date(2017, 11, 27): {'winner': 'SAC', 'loser': 'GSW', 'winner_val': 0.5789433333333334, 'loser_val': 0.5133305628552222}, datetime.date(2017, 11, 28): {'winner': 'UTA', 'loser': 'DEN', 'winner_val': 0.6788820360683895, 'loser_val': 0.5239761555075593}, datetime.date(2017, 11, 29): {'winner': 'BKN', 'loser': 'DAL', 'winner_val': 0.43546427986236275, 'loser_val': 0.4517409798708697}, datetime.date(2017, 11, 30): {'winner': 'MIL', 'loser': 'POR', 'winner_val': 0.5635191657271702, 'loser_val': 0.5267293605624374}, datetime.date(2017, 12, 1): {'winner': 'UTA', 'loser': 'NOP', 'winner_val': 0.6788820360683895, 'loser_val': 0.5740940719606911}, datetime.date(2017, 12, 2): {'winner': 'NOP', 'loser': 'POR', 'winner_val': 0.5740940719606911, 'loser_val': 0.5267293605624374}, datetime.date(2017, 12, 4): {'winner': 'MEM', 'loser': 'MIN', 'winner_val': 0.6142030394640086, 'loser_val': 0.6258844447843378}, datetime.date(2017, 12, 5): {'winner': 'WAS', 'loser': 'POR', 'winner_val': 0.6554382834390208, 'loser_val': 0.5267293605624374}, datetime.date(2017, 12, 7): {'winner': 'BKN', 'loser': 'OKC', 'winner_val': 0.43546427986236275, 'loser_val': 0.674630264717503}, datetime.date(2017, 12, 8): {'winner': 'SAC', 'loser': 'NOP', 'winner_val': 0.5789433333333334, 'loser_val': 0.5740940719606911}, datetime.date(2017, 12, 9): {'winner': 'MIL', 'loser': 'UTA', 'winner_val': 0.5635191657271702, 'loser_val': 0.6788820360683895}, datetime.date(2017, 12, 11): {'winner': 'LAC', 'loser': 'TOR', 'winner_val': 0.6419989164086688, 'loser_val': 0.6292524014336919}, datetime.date(2017, 12, 12): {'winner': 'DAL', 'loser': 'SAS', 'winner_val': 0.4517409798708697, 'loser_val': 0.5954422674529198}, datetime.date(2017, 12, 13): {'winner': 'CHI', 'loser': 'UTA', 'winner_val': 0.6694730009096169, 'loser_val': 0.6788820360683895}, datetime.date(2017, 12, 14): {'winner': 'NYK', 'loser': 'BKN', 'winner_val': 0.5721603765993403, 'loser_val': 0.43546427986236275}, datetime.date(2017, 12, 15): {'winner': 'CHI', 'loser': 'MIL', 'winner_val': 0.6694730009096169, 'loser_val': 0.5635191657271702}, datetime.date(2017, 12, 16): {'winner': 'PHX', 'loser': 'MIN', 'winner_val': 0.5236721281795963, 'loser_val': 0.6258844447843378}, datetime.date(2017, 12, 17): {'winner': 'CLE', 'loser': 'WAS', 'winner_val': 0.5390168074699865, 'loser_val': 0.6554382834390208}, datetime.date(2017, 12, 18): {'winner': 'PHX', 'loser': 'DAL', 'winner_val': 0.5236721281795963, 'loser_val': 0.4517409798708697}, datetime.date(2017, 12, 19): {'winner': 'MIL', 'loser': 'CLE', 'winner_val': 0.5635191657271702, 'loser_val': 0.5390168074699865}, datetime.date(2017, 12, 20): {'winner': 'SAS', 'loser': 'POR', 'winner_val': 0.5954422674529198, 'loser_val': 0.5267293605624374}, datetime.date(2017, 12, 21): {'winner': 'UTA', 'loser': 'SAS', 'winner_val': 0.6788820360683895, 'loser_val': 0.5954422674529198}, datetime.date(2017, 12, 22): {'winner': 'LAC', 'loser': 'HOU', 'winner_val': 0.6419989164086688, 'loser_val': 0.6132173179118474}, datetime.date(2017, 12, 23): {'winner': 'POR', 'loser': 'LAL', 'winner_val': 0.5267293605624374, 'loser_val': 0.5261833304602654}, datetime.date(2017, 12, 25): {'winner': 'WAS', 'loser': 'BOS', 'winner_val': 0.6554382834390208, 'loser_val': 0.6228089082081968}, datetime.date(2017, 12, 26): {'winner': 'PHX', 'loser': 'MEM', 'winner_val': 0.5236721281795963, 'loser_val': 0.6142030394640086}, datetime.date(2017, 12, 27): {'winner': 'MEM', 'loser': 'LAL', 'winner_val': 0.6142030394640086, 'loser_val': 0.5261833304602654}, datetime.date(2017, 12, 28): {'winner': 'POR', 'loser': 'PHI', 'winner_val': 0.5267293605624374, 'loser_val': 0.6389277889214628}, datetime.date(2017, 12, 29): {'winner': 'CHA', 'loser': 'GSW', 'winner_val': 0.5336705177879535, 'loser_val': 0.5133305628552222}, datetime.date(2017, 12, 30): {'winner': 'PHI', 'loser': 'DEN', 'winner_val': 0.6389277889214628, 'loser_val': 0.5239761555075593}, datetime.date(2017, 12, 31): {'winner': 'MEM', 'loser': 'SAC', 'winner_val': 0.6142030394640086, 'loser_val': 0.5789433333333334}, datetime.date(2018, 1, 1): {'winner': 'POR', 'loser': 'CHI', 'winner_val': 0.5267293605624374, 'loser_val': 0.6694730009096169}, datetime.date(2018, 1, 2): {'winner': 'PHX', 'loser': 'ATL', 'winner_val': 0.5236721281795963, 'loser_val': 0.5618165103879389}, datetime.date(2018, 1, 3): {'winner': 'BKN', 'loser': 'MIN', 'winner_val': 0.43546427986236275, 'loser_val': 0.6258844447843378}, datetime.date(2018, 1, 5): {'winner': 'CHI', 'loser': 'DAL', 'winner_val': 0.6694730009096169, 'loser_val': 0.4517409798708697}, datetime.date(2018, 1, 6): {'winner': 'SAC', 'loser': 'DEN', 'winner_val': 0.5789433333333334, 'loser_val': 0.5239761555075593}, datetime.date(2018, 1, 7): {'winner': 'PHX', 'loser': 'OKC', 'winner_val': 0.5236721281795963, 'loser_val': 0.674630264717503}, datetime.date(2018, 1, 8): {'winner': 'MIN', 'loser': 'CLE', 'winner_val': 0.6258844447843378, 'loser_val': 0.5390168074699865}, datetime.date(2018, 1, 9): {'winner': 'POR', 'loser': 'OKC', 'winner_val': 0.5267293605624374, 'loser_val': 0.674630264717503}, datetime.date(2018, 1, 10): {'winner': 'LAC', 'loser': 'GSW', 'winner_val': 0.6419989164086688, 'loser_val': 0.5133305628552222}, datetime.date(2018, 1, 11): {'winner': 'LAL', 'loser': 'SAS', 'winner_val': 0.5261833304602654, 'loser_val': 0.5954422674529198}, datetime.date(2018, 1, 12): {'winner': 'BKN', 'loser': 'ATL', 'winner_val': 0.43546427986236275, 'loser_val': 0.5618165103879389}, datetime.date(2018, 1, 13): {'winner': 'CHI', 'loser': 'DET', 'winner_val': 0.6694730009096169, 'loser_val': 0.632708410067526}, datetime.date(2018, 1, 15): {'winner': 'LAC', 'loser': 'HOU', 'winner_val': 0.6419989164086688, 'loser_val': 0.6132173179118474}, datetime.date(2018, 1, 16): {'winner': 'NOP', 'loser': 'BOS', 'winner_val': 0.5740940719606911, 'loser_val': 0.6228089082081968}, datetime.date(2018, 1, 17): {'winner': 'MEM', 'loser': 'NYK', 'winner_val': 0.6142030394640086, 'loser_val': 0.5721603765993403}, datetime.date(2018, 1, 18): {'winner': 'PHI', 'loser': 'BOS', 'winner_val': 0.6389277889214628, 'loser_val': 0.6228089082081968}, datetime.date(2018, 1, 19): {'winner': 'LAL', 'loser': 'IND', 'winner_val': 0.5261833304602654, 'loser_val': 0.668143035465159}, datetime.date(2018, 1, 20): {'winner': 'MIN', 'loser': 'TOR', 'winner_val': 0.6258844447843378, 'loser_val': 0.6292524014336919}, datetime.date(2018, 1, 21): {'winner': 'IND', 'loser': 'SAS', 'winner_val': 0.668143035465159, 'loser_val': 0.5954422674529198}, datetime.date(2018, 1, 22): {'winner': 'MIN', 'loser': 'LAC', 'winner_val': 0.6258844447843378, 'loser_val': 0.6419989164086688}, datetime.date(2018, 1, 23): {'winner': 'LAL', 'loser': 'BOS', 'winner_val': 0.5261833304602654, 'loser_val': 0.6228089082081968}, datetime.date(2018, 1, 24): {'winner': 'BOS', 'loser': 'LAC', 'winner_val': 0.6228089082081968, 'loser_val': 0.6419989164086688}, datetime.date(2018, 1, 25): {'winner': 'SAC', 'loser': 'MIA', 'winner_val': 0.5789433333333334, 'loser_val': 0.568429169249845}, datetime.date(2018, 1, 26): {'winner': 'PHI', 'loser': 'SAS', 'winner_val': 0.6389277889214628, 'loser_val': 0.5954422674529198}, datetime.date(2018, 1, 28): {'winner': 'LAC', 'loser': 'NOP', 'winner_val': 0.6419989164086688, 'loser_val': 0.5740940719606911}, datetime.date(2018, 1, 29): {'winner': 'BOS', 'loser': 'DEN', 'winner_val': 0.6228089082081968, 'loser_val': 0.5239761555075593}, datetime.date(2018, 1, 30): {'winner': 'UTA', 'loser': 'GSW', 'winner_val': 0.6788820360683895, 'loser_val': 0.5133305628552222}, datetime.date(2018, 1, 31): {'winner': 'PHX', 'loser': 'DAL', 'winner_val': 0.5236721281795963, 'loser_val': 0.4517409798708697}, datetime.date(2018, 2, 1): {'winner': 'DEN', 'loser': 'OKC', 'winner_val': 0.5239761555075593, 'loser_val': 0.674630264717503}, datetime.date(2018, 2, 2): {'winner': 'NOP', 'loser': 'OKC', 'winner_val': 0.5740940719606911, 'loser_val': 0.674630264717503}, datetime.date(2018, 2, 3): {'winner': 'DEN', 'loser': 'GSW', 'winner_val': 0.5239761555075593, 'loser_val': 0.5133305628552222}, datetime.date(2018, 2, 4): {'winner': 'LAL', 'loser': 'OKC', 'winner_val': 0.5261833304602654, 'loser_val': 0.674630264717503}, datetime.date(2018, 2, 5): {'winner': 'ORL', 'loser': 'MIA', 'winner_val': 0.5711635006221484, 'loser_val': 0.568429169249845}, datetime.date(2018, 2, 6): {'winner': 'OKC', 'loser': 'GSW', 'winner_val': 0.674630264717503, 'loser_val': 0.5133305628552222}, datetime.date(2018, 2, 7): {'winner': 'CLE', 'loser': 'MIN', 'winner_val': 0.5390168074699865, 'loser_val': 0.6258844447843378}, datetime.date(2018, 2, 8): {'winner': 'ORL', 'loser': 'ATL', 'winner_val': 0.5711635006221484, 'loser_val': 0.5618165103879389}, datetime.date(2018, 2, 9): {'winner': 'CHI', 'loser': 'MIN', 'winner_val': 0.6694730009096169, 'loser_val': 0.6258844447843378}, datetime.date(2018, 2, 11): {'winner': 'UTA', 'loser': 'POR', 'winner_val': 0.6788820360683895, 'loser_val': 0.5267293605624374}, datetime.date(2018, 2, 12): {'winner': 'NOP', 'loser': 'DET', 'winner_val': 0.5740940719606911, 'loser_val': 0.632708410067526}, datetime.date(2018, 2, 13): {'winner': 'SAC', 'loser': 'DAL', 'winner_val': 0.5789433333333334, 'loser_val': 0.4517409798708697}, datetime.date(2018, 2, 14): {'winner': 'POR', 'loser': 'GSW', 'winner_val': 0.5267293605624374, 'loser_val': 0.5133305628552222}, datetime.date(2018, 2, 15): {'winner': 'DEN', 'loser': 'MIL', 'winner_val': 0.5239761555075593, 'loser_val': 0.5635191657271702}, datetime.date(2018, 2, 22): {'winner': 'WAS', 'loser': 'CLE', 'winner_val': 0.6554382834390208, 'loser_val': 0.5390168074699865}, datetime.date(2018, 2, 23): {'winner': 'POR', 'loser': 'UTA', 'winner_val': 0.5267293605624374, 'loser_val': 0.6788820360683895}, datetime.date(2018, 2, 25): {'winner': 'SAS', 'loser': 'CLE', 'winner_val': 0.5954422674529198, 'loser_val': 0.5390168074699865}, datetime.date(2018, 2, 26): {'winner': 'DAL', 'loser': 'IND', 'winner_val': 0.4517409798708697, 'loser_val': 0.668143035465159}, datetime.date(2018, 2, 27): {'winner': 'LAC', 'loser': 'DEN', 'winner_val': 0.6419989164086688, 'loser_val': 0.5239761555075593}, datetime.date(2018, 2, 28): {'winner': 'NOP', 'loser': 'SAS', 'winner_val': 0.5740940719606911, 'loser_val': 0.5954422674529198}, datetime.date(2018, 3, 1): {'winner': 'SAC', 'loser': 'BKN', 'winner_val': 0.5789433333333334, 'loser_val': 0.43546427986236275}, datetime.date(2018, 3, 2): {'winner': 'IND', 'loser': 'MIL', 'winner_val': 0.668143035465159, 'loser_val': 0.5635191657271702}, datetime.date(2018, 3, 3): {'winner': 'LAL', 'loser': 'SAS', 'winner_val': 0.5261833304602654, 'loser_val': 0.5954422674529198}, datetime.date(2018, 3, 4): {'winner': 'SAC', 'loser': 'NYK', 'winner_val': 0.5789433333333334, 'loser_val': 0.5721603765993403}, datetime.date(2018, 3, 6): {'winner': 'NOP', 'loser': 'LAC', 'winner_val': 0.5740940719606911, 'loser_val': 0.6419989164086688}, datetime.date(2018, 3, 7): {'winner': 'CLE', 'loser': 'DEN', 'winner_val': 0.5390168074699865, 'loser_val': 0.5239761555075593}, datetime.date(2018, 3, 8): {'winner': 'MIA', 'loser': 'PHI', 'winner_val': 0.568429169249845, 'loser_val': 0.6389277889214628}, datetime.date(2018, 3, 9): {'winner': 'LAC', 'loser': 'CLE', 'winner_val': 0.6419989164086688, 'loser_val': 0.5390168074699865}, datetime.date(2018, 3, 11): {'winner': 'LAL', 'loser': 'CLE', 'winner_val': 0.5261833304602654, 'loser_val': 0.5390168074699865}, datetime.date(2018, 3, 13): {'winner': 'LAL', 'loser': 'DEN', 'winner_val': 0.5261833304602654, 'loser_val': 0.5239761555075593}, datetime.date(2018, 3, 14): {'winner': 'SAC', 'loser': 'MIA', 'winner_val': 0.5789433333333334, 'loser_val': 0.568429169249845}, datetime.date(2018, 3, 15): {'winner': 'CHI', 'loser': 'MEM', 'winner_val': 0.6694730009096169, 'loser_val': 0.6142030394640086}, datetime.date(2018, 3, 16): {'winner': 'MIA', 'loser': 'LAL', 'winner_val': 0.568429169249845, 'loser_val': 0.5261833304602654}, datetime.date(2018, 3, 17): {'winner': 'MEM', 'loser': 'DEN', 'winner_val': 0.6142030394640086, 'loser_val': 0.5239761555075593}, datetime.date(2018, 3, 18): {'winner': 'POR', 'loser': 'LAC', 'winner_val': 0.5267293605624374, 'loser_val': 0.6419989164086688}, datetime.date(2018, 3, 20): {'winner': 'ATL', 'loser': 'UTA', 'winner_val': 0.5618165103879389, 'loser_val': 0.6788820360683895}, datetime.date(2018, 3, 21): {'winner': 'LAC', 'loser': 'MIL', 'winner_val': 0.6419989164086688, 'loser_val': 0.5635191657271702}, datetime.date(2018, 3, 23): {'winner': 'BOS', 'loser': 'POR', 'winner_val': 0.6228089082081968, 'loser_val': 0.5267293605624374}, datetime.date(2018, 3, 25): {'winner': 'POR', 'loser': 'OKC', 'winner_val': 0.5267293605624374, 'loser_val': 0.674630264717503}, datetime.date(2018, 3, 26): {'winner': 'MEM', 'loser': 'MIN', 'winner_val': 0.6142030394640086, 'loser_val': 0.6258844447843378}, datetime.date(2018, 3, 27): {'winner': 'DAL', 'loser': 'SAC', 'winner_val': 0.4517409798708697, 'loser_val': 0.5789433333333334}, datetime.date(2018, 3, 28): {'winner': 'BOS', 'loser': 'UTA', 'winner_val': 0.6228089082081968, 'loser_val': 0.6788820360683895}, datetime.date(2018, 3, 29): {'winner': 'MIL', 'loser': 'GSW', 'winner_val': 0.5635191657271702, 'loser_val': 0.5133305628552222}, datetime.date(2018, 3, 30): {'winner': 'DEN', 'loser': 'OKC', 'winner_val': 0.5239761555075593, 'loser_val': 0.674630264717503}, datetime.date(2018, 3, 31): {'winner': 'BKN', 'loser': 'MIA', 'winner_val': 0.43546427986236275, 'loser_val': 0.568429169249845}, datetime.date(2018, 4, 1): {'winner': 'SAC', 'loser': 'LAL', 'winner_val': 0.5789433333333334, 'loser_val': 0.5261833304602654}, datetime.date(2018, 4, 3): {'winner': 'LAC', 'loser': 'SAS', 'winner_val': 0.6419989164086688, 'loser_val': 0.5954422674529198}, datetime.date(2018, 4, 4): {'winner': 'LAL', 'loser': 'SAS', 'winner_val': 0.5261833304602654, 'loser_val': 0.5954422674529198}, datetime.date(2018, 4, 5): {'winner': 'BKN', 'loser': 'MIL', 'winner_val': 0.43546427986236275, 'loser_val': 0.5635191657271702}, datetime.date(2018, 4, 6): {'winner': 'NYK', 'loser': 'MIA', 'winner_val': 0.5721603765993403, 'loser_val': 0.568429169249845}, datetime.date(2018, 4, 7): {'winner': 'OKC', 'loser': 'HOU', 'winner_val': 0.674630264717503, 'loser_val': 0.6132173179118474}, datetime.date(2018, 4, 8): {'winner': 'MEM', 'loser': 'DET', 'winner_val': 0.6142030394640086, 'loser_val': 0.632708410067526}, datetime.date(2018, 4, 10): {'winner': 'PHX', 'loser': 'DAL', 'winner_val': 0.5236721281795963, 'loser_val': 0.4517409798708697}, datetime.date(2018, 4, 11): {'winner': 'LAL', 'loser': 'LAC', 'winner_val': 0.5261833304602654, 'loser_val': 0.6419989164086688}, datetime.date(2018, 4, 14): {'winner': 'NOP', 'loser': 'POR', 'winner_val': 0.5740940719606911, 'loser_val': 0.5267293605624374}, datetime.date(2018, 4, 15): {'winner': 'IND', 'loser': 'CLE', 'winner_val': 0.668143035465159, 'loser_val': 0.5390168074699865}, datetime.date(2018, 4, 16): {'winner': 'MIA', 'loser': 'PHI', 'winner_val': 0.568429169249845, 'loser_val': 0.6389277889214628}, datetime.date(2018, 4, 17): {'winner': 'NOP', 'loser': 'POR', 'winner_val': 0.5740940719606911, 'loser_val': 0.5267293605624374}, datetime.date(2018, 4, 18): {'winner': 'UTA', 'loser': 'OKC', 'winner_val': 0.6788820360683895, 'loser_val': 0.674630264717503}, datetime.date(2018, 4, 20): {'winner': 'WAS', 'loser': 'TOR', 'winner_val': 0.6554382834390208, 'loser_val': 0.6292524014336919}, datetime.date(2018, 4, 21): {'winner': 'MIN', 'loser': 'HOU', 'winner_val': 0.6258844447843378, 'loser_val': 0.6132173179118474}, datetime.date(2018, 4, 22): {'winner': 'WAS', 'loser': 'TOR', 'winner_val': 0.6554382834390208, 'loser_val': 0.6292524014336919}, datetime.date(2018, 4, 30): {'winner': 'BOS', 'loser': 'PHI', 'winner_val': 0.6228089082081968, 'loser_val': 0.6389277889214628}, datetime.date(2018, 5, 1): {'winner': 'CLE', 'loser': 'TOR', 'winner_val': 0.5390168074699865, 'loser_val': 0.6292524014336919}, datetime.date(2018, 5, 2): {'winner': 'UTA', 'loser': 'HOU', 'winner_val': 0.6788820360683895, 'loser_val': 0.6132173179118474}, datetime.date(2018, 5, 3): {'winner': 'CLE', 'loser': 'TOR', 'winner_val': 0.5390168074699865, 'loser_val': 0.6292524014336919}, datetime.date(2018, 5, 4): {'winner': 'NOP', 'loser': 'GSW', 'winner_val': 0.5740940719606911, 'loser_val': 0.5133305628552222}, datetime.date(2018, 5, 5): {'winner': 'BOS', 'loser': 'PHI', 'winner_val': 0.6228089082081968, 'loser_val': 0.6389277889214628}, datetime.date(2018, 5, 13): {'winner': 'BOS', 'loser': 'CLE', 'winner_val': 0.6228089082081968, 'loser_val': 0.5390168074699865}, datetime.date(2018, 5, 14): {'winner': 'GSW', 'loser': 'HOU', 'winner_val': 0.5133305628552222, 'loser_val': 0.6132173179118474}, datetime.date(2018, 5, 15): {'winner': 'BOS', 'loser': 'CLE', 'winner_val': 0.6228089082081968, 'loser_val': 0.5390168074699865}, datetime.date(2018, 5, 22): {'winner': 'HOU', 'loser': 'GSW', 'winner_val': 0.6132173179118474, 'loser_val': 0.5133305628552222}, datetime.date(2018, 5, 24): {'winner': 'HOU', 'loser': 'GSW', 'winner_val': 0.6132173179118474, 'loser_val': 0.5133305628552222}, datetime.date(2018, 5, 27): {'winner': 'CLE', 'loser': 'BOS', 'winner_val': 0.5390168074699865, 'loser_val': 0.6228089082081968}}

 
Let's save this data for the future. 

**In [9]:**

{% highlight python %}
with open(f'perimeter_defense_{YEAR}.csv', 'w') as f:
    f.write('date,winner,loser,winner_score,loser_score\n')
    for d in data:
        date = d.strftime('%Y-%m-%d')
        f.write(f"{date},{data[d]['winner']},{data[d]['loser']},{data[d]['winner_val']},{data[d]['loser_val']}\n")
{% endhighlight %}
 
## Visualization

I graphed the difference between the winning team's score and the losing team's
score for each year and calculated the number of *positive* differences and
*negative* differences. *Positive* differences indicate that the winning team's
score was greater than the losing team's score, even though Westgate predicted
the losing team would win. Hence, a greater number of *positive* differences
indicates that Perimeter Defense is a market inefficiency. Obviously, the same
applies for *negative* differences. 

**In [10]:**

{% highlight python %}
from bokeh.plotting import figure, output_notebook, show
from bokeh.io import export_png
from bokeh.models import Label, Title
import math

output_notebook()
def draw_chart(data, fname):
    x = []
    y = []
    seg1 = []
    seg2 = []
    colors = []
    aboveName = []
    belowName = []
    for d in data:
        date = d.strftime('%m/%d/%y')
        x.append(f"{date} ({data[d]['winner']} vs. {data[d]['loser']})")
        if data[d]['winner_val']>data[d]['loser_val']:
            colors.append('green')
            aboveName.append(data[d]['winner'])
            belowName.append(data[d]['loser'])
            seg1.append(data[d]['winner_val'])
            seg2.append(data[d]['loser_val'])
        else:
            colors.append('red')
            aboveName.append(data[d]['loser'])
            belowName.append(data[d]['winner'])
            seg2.append(data[d]['winner_val'])
            seg1.append(data[d]['loser_val'])


    p = figure(x_range=x, plot_width=800, plot_height=400, title='')
    p.xaxis.major_label_orientation = math.pi/2
    p.xaxis.axis_label = 'Westgate Incorrectly Predicted Game'
    p.yaxis.axis_label = 'Average Perimeter Defensive Score'
    for s in range(0, len(seg1)):
        p.segment(y0=seg1[s], y1=seg1[s], x0=s+0.25, x1=s+0.75, color=colors[s])
    for s in range(0, len(seg2)):
        p.segment(y0=seg2[s], y1=seg2[s], x0=s+0.25, x1=s+0.75, color=colors[s])
    for s in range(0, len(seg2)):
        p.segment(y0=seg1[s], y1=seg2[s], x0=s+0.5, x1=s+0.5, color=colors[s])
    for s in range(0, len(aboveName)):
        l = Label(x=s+0.25, y=seg1[s], text=aboveName[s])
        p.add_layout(l)
    for s in range(0, len(belowName)):
        l = Label(x=s+0.25, y=seg2[s]-0.02, text=belowName[s])
        p.add_layout(l)
    export_png(p, filename=f"{YEAR}_{fname}.png")
    show(p)
d2 = {}
fname = 1
for d in data:
    d2[d] = data[d]
    if len(d2.keys())>10:
        draw_chart(d2, fname)
        fname+=1
        d2 = {}
{% endhighlight %}


![img](https://i.imgur.com/sviVpAe.png)
![img](https://i.imgur.com/Zuxqi0c.png)
![img](https://i.imgur.com/MgtCGFh.png)
![img](https://i.imgur.com/0k1msQO.png)
![img](https://i.imgur.com/iOv9Lyl.png)
![img](https://i.imgur.com/gjl3O99.png)
![img](https://i.imgur.com/v6lCV3k.png)
![img](https://i.imgur.com/8IVnJhZ.png)
![img](https://i.imgur.com/zjDMkg8.png)
![img](https://i.imgur.com/yLVRP0y.png)
![img](https://i.imgur.com/qHx4GZ9.png)
![img](https://i.imgur.com/OCWuCxP.png)
![img](https://i.imgur.com/JHKsnEh.png)
![img](https://i.imgur.com/eoxGqOF.png)
![img](https://i.imgur.com/ib9pYFl.png)


**In [11]:**

{% highlight python %}
def draw_chart2(data):
    x = []
    y = []
    seg = []
    pos_count = 0
    neg_count = 0
    for d in data:
        date = d.strftime('%m/%d/%y')
        x.append(f"{date} ({data[d]['winner']} vs. {data[d]['loser']})")
        seg.append(data[d]['winner_val']-data[d]['loser_val'])
        if seg[-1]>0:
            pos_count+=1
        else:
            neg_count+=1
    p = figure(x_range=x, plot_width=800, plot_height=400, title=f'Difference in Perimeter Defense in Incorrectly Predicted Westgate {YEAR} NBA Regular Season Games')
    p.xaxis.major_label_text_font_size = '0pt'
    p.xaxis.major_tick_line_color = None  # turn off x-axis major ticks
    p.xaxis.minor_tick_line_color = None  # turn off x-axis minor ticks
    p.xaxis.major_label_text_color = None  # turn off x-axis tick labels leaving space
    p.xaxis.major_label_orientation = math.pi/2
    p.xaxis.axis_label = 'Westgate Incorrectly Predicted Games'
    p.yaxis.axis_label = 'Difference in Average Perimeter Defensive Score'
    for s in range(0, len(seg)):
        if seg[s]<0:
            color = 'red'
        else:
            color = 'green'
        p.segment(y0=0, y1=seg[s], x0=s+0.5, x1=s+0.5, color=color)
    p.add_layout(Title(text=f"Positive Differences: {pos_count}, Negative Differences {neg_count}", align="center"), "below")
    export_png(p, filename=f"{YEAR}.png")
    show(p)
    
draw_chart2(data)
{% endhighlight %}

![img](https://i.imgur.com/EOnwEq8.png)
![img](https://i.imgur.com/ObtfNig.png)
![img](https://i.imgur.com/0E1MLdq.png)
![img](https://i.imgur.com/0rwYwtk.png)
![img](https://i.imgur.com/cJB0QYR.png)
