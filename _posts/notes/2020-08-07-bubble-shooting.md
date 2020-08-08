---
title: "Differences in FG% and 3P% Shooting in the NBA Bubble - Small sample size"
date: 2020-08-07
permalink: /notes/2020/08/07/bubble-shooting
tags:
    - python    
    - - notebook
--- 
# Do teams shoot better or worse in the NBA bubble? 
 
First, let's get field goal % and three point % before the bubble. This will
require the following steps:

1) Get the season schedule

2) Get the box score for every single game

3) For each game, get FGM, FGA, 3PM, 3PA 
 
We'll use a modified version from the `basketball_reference_scaper`'s
`get_schedule` function.

Normally, the `months` variable doesn't include July or August, but let's modify
it so it does. 

**In [15]:**

{% highlight python %}
import pandas as pd
from datetime import datetime
from requests import get
from bs4 import BeautifulSoup

def get_schedule(season, playoffs=False):
    months = ['October', 'November', 'December', 'January', 'February', 'March',
            'July', 'August']
    df = pd.DataFrame()
    for month in months:
        r = get(f'https://www.basketball-reference.com/leagues/NBA_{season}_games-{month.lower()}.html')
        if r.status_code==200:
            soup = BeautifulSoup(r.content, 'html.parser')
            table = soup.find('table', attrs={'id': 'schedule'})
            month_df = pd.read_html(str(table))[0]
            df = df.append(month_df)
    df = df.reset_index()
    cols_to_remove = [i for i in df.columns if 'Unnamed' in i]
    cols_to_remove += [i for i in df.columns if 'Notes' in i]
    cols_to_remove += [i for i in df.columns if 'Start' in i]
    cols_to_remove += [i for i in df.columns if 'Attend' in i]
    cols_to_remove += ['index']
    df = df.drop(cols_to_remove, axis=1)
    df.columns = ['DATE', 'VISITOR', 'VISITOR_PTS', 'HOME', 'HOME_PTS']
    playoff_loc = df[df['DATE']=='Playoffs']
    if len(playoff_loc.index)>0:
        playoff_index = playoff_loc.index[0]
    else:
        playoff_index = len(df)
    if playoffs:
        df = df[playoff_index+1:]
    else:
        df = df[:playoff_index]
    df['DATE'] = df['DATE'].apply(lambda x: pd.to_datetime(x))
    return df

def get_standings(date=None):
    if date is None:
        date = datetime.now()
    else:
        date = pd.to_datetime(date)
    d = {}
    r = get(f'https://www.basketball-reference.com/friv/standings.fcgi?month={date.month}&day={date.day}&year={date.year}')
    if r.status_code==200:
        soup = BeautifulSoup(r.content, 'html.parser')
        e_table = soup.find('table', attrs={'id': 'standings_e'})
        e_df = pd.read_html(str(e_table))[0]
        w_table = soup.find('table', attrs={'id': 'standings_w'})
        w_df = pd.read_html(str(w_table))[0]
        e_df.rename(columns={'Eastern Conference': 'TEAM'}, inplace=True)
        w_df.rename(columns={'Western Conference': 'TEAM'}, inplace=True)
        d['EASTERN_CONF'] = e_df
        d['WESTERN_CONF'] = w_df
    return d
{% endhighlight %}

**In [16]:**

{% highlight python %}
schedule = get_schedule(2020)
schedule
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
      <th>DATE</th>
      <th>VISITOR</th>
      <th>VISITOR_PTS</th>
      <th>HOME</th>
      <th>HOME_PTS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-10-22</td>
      <td>New Orleans Pelicans</td>
      <td>122.0</td>
      <td>Toronto Raptors</td>
      <td>130.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-10-22</td>
      <td>Los Angeles Lakers</td>
      <td>102.0</td>
      <td>Los Angeles Clippers</td>
      <td>112.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-10-23</td>
      <td>Chicago Bulls</td>
      <td>125.0</td>
      <td>Charlotte Hornets</td>
      <td>126.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-10-23</td>
      <td>Detroit Pistons</td>
      <td>119.0</td>
      <td>Indiana Pacers</td>
      <td>110.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-10-23</td>
      <td>Cleveland Cavaliers</td>
      <td>85.0</td>
      <td>Orlando Magic</td>
      <td>94.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1054</th>
      <td>2020-08-13</td>
      <td>Portland Trail Blazers</td>
      <td>NaN</td>
      <td>Brooklyn Nets</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1055</th>
      <td>2020-08-14</td>
      <td>Philadelphia 76ers</td>
      <td>NaN</td>
      <td>Houston Rockets</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1056</th>
      <td>2020-08-14</td>
      <td>Miami Heat</td>
      <td>NaN</td>
      <td>Indiana Pacers</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1057</th>
      <td>2020-08-14</td>
      <td>Oklahoma City Thunder</td>
      <td>NaN</td>
      <td>Los Angeles Clippers</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1058</th>
      <td>2020-08-14</td>
      <td>Denver Nuggets</td>
      <td>NaN</td>
      <td>Toronto Raptors</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>1059 rows × 5 columns</p>
</div>


 
Now, we can acquire the stats for every team game prior to the break. 

**In [17]:**

{% highlight python %}
from basketball_reference_scraper.box_scores import get_box_scores
def get_game_stats(date, t1, t2):
    box_score = get_box_scores(date, t1, t2)
    if t1 not in before:
        before[t1] = {'FG': 0, 'FGA': 0, '3P': 0, '3PA': 0}
    if t2 not in before:
        before[t2] = {'FG': 0, 'FGA': 0, '3P': 0, '3PA': 0}
    before[t1]['FG']+=int(box_score[t1]['FG'].values[-1])
    before[t1]['FGA']+=int(box_score[t1]['FGA'].values[-1])
    before[t1]['3P']+=int(box_score[t1]['3P'].values[-1])
    before[t1]['3PA']+=int(box_score[t1]['3PA'].values[-1])
    before[t2]['FG']+=int(box_score[t2]['FG'].values[-1])
    before[t2]['FGA']+=int(box_score[t2]['FGA'].values[-1])
    before[t2]['3P']+=int(box_score[t2]['3P'].values[-1])
    before[t2]['3PA']+=int(box_score[t2]['3PA'].values[-1])
{% endhighlight %}

**In [18]:**

{% highlight python %}
from basketball_reference_scraper.constants import TEAM_TO_TEAM_ABBR
before_written = True
before = {}
for i, r in schedule.iterrows():
    if r['DATE'].month==7 or r['DATE'].month==8 or before_written:
        break
    print(r['DATE'])
    t1 = TEAM_TO_TEAM_ABBR[r['HOME'].upper()]
    t2 = TEAM_TO_TEAM_ABBR[r['VISITOR'].upper()]
    date = r['DATE'].strftime('%Y-%m-%d')
    get_game_stats(date, t1, t2)
    
if before_written:
    before = pd.read_csv('before.csv')
{% endhighlight %}

**In [19]:**

{% highlight python %}
before = before.set_index('Unnamed: 0')
{% endhighlight %}

**In [20]:**

{% highlight python %}
if not before_written:
    for t in before:
        before[t]['FG%'] = before[t]['FG']/float(before[t]['FGA'])
        before[t]['3P%'] = before[t]['3P']/float(before[t]['3PA'])
{% endhighlight %}
 
Now we can acquire all the games during the bubble era: 

**In [21]:**

{% highlight python %}
def get_game_stats_2(date, t1, t2):
    box_score = get_box_scores(date, t1, t2)
    if t1 not in after:
        after[t1] = {'FG': 0, 'FGA': 0, '3P': 0, '3PA': 0}
    if t2 not in after:
        after[t2] = {'FG': 0, 'FGA': 0, '3P': 0, '3PA': 0}
    after[t1]['FG']+=int(box_score[t1]['FG'].values[-1])
    after[t1]['FGA']+=int(box_score[t1]['FGA'].values[-1])
    after[t1]['3P']+=int(box_score[t1]['3P'].values[-1])
    after[t1]['3PA']+=int(box_score[t1]['3PA'].values[-1])
    after[t2]['FG']+=int(box_score[t2]['FG'].values[-1])
    after[t2]['FGA']+=int(box_score[t2]['FGA'].values[-1])
    after[t2]['3P']+=int(box_score[t2]['3P'].values[-1])
    after[t2]['3PA']+=int(box_score[t2]['3PA'].values[-1])
{% endhighlight %}

**In [22]:**

{% highlight python %}
after = {}
now = pd.Timestamp.now().strftime('%Y-%m-%d')
for i, r in schedule.iterrows():
    if r['DATE'].month==7 or r['DATE'].month==8:
        print(r['DATE'])
        t1 = TEAM_TO_TEAM_ABBR[r['HOME'].upper()]
        t2 = TEAM_TO_TEAM_ABBR[r['VISITOR'].upper()]
        date = r['DATE'].strftime('%Y-%m-%d')
        if date==now:
            break
        get_game_stats_2(date, t1, t2)
{% endhighlight %}

    2020-07-30 00:00:00
    2020-07-30 00:00:00
    2020-07-31 00:00:00
    2020-07-31 00:00:00
    2020-07-31 00:00:00
    2020-07-31 00:00:00
    2020-07-31 00:00:00
    2020-07-31 00:00:00
    2020-08-01 00:00:00
    2020-08-01 00:00:00
    2020-08-01 00:00:00
    2020-08-01 00:00:00
    2020-08-01 00:00:00
    2020-08-02 00:00:00
    2020-08-02 00:00:00
    2020-08-02 00:00:00
    2020-08-02 00:00:00
    2020-08-02 00:00:00
    2020-08-02 00:00:00
    2020-08-03 00:00:00
    2020-08-03 00:00:00
    2020-08-03 00:00:00
    2020-08-03 00:00:00
    2020-08-03 00:00:00
    2020-08-03 00:00:00
    2020-08-04 00:00:00
    2020-08-04 00:00:00
    2020-08-04 00:00:00
    2020-08-04 00:00:00
    2020-08-04 00:00:00
    2020-08-04 00:00:00
    2020-08-05 00:00:00
    2020-08-05 00:00:00
    2020-08-05 00:00:00
    2020-08-05 00:00:00
    2020-08-05 00:00:00
    2020-08-05 00:00:00
    2020-08-06 00:00:00
    2020-08-06 00:00:00
    2020-08-06 00:00:00
    2020-08-06 00:00:00
    2020-08-06 00:00:00
    2020-08-06 00:00:00
    2020-08-07 00:00:00


**In [23]:**

{% highlight python %}
after
{% endhighlight %}




    {'NOP': {'FG': 162, 'FGA': 343, '3P': 45, '3PA': 124},
     'UTA': {'FG': 146, 'FGA': 338, '3P': 46, '3PA': 153},
     'LAL': {'FG': 168, 'FGA': 412, '3P': 37, '3PA': 158},
     'LAC': {'FG': 161, 'FGA': 338, '3P': 63, '3PA': 143},
     'BRK': {'FG': 170, 'FGA': 379, '3P': 53, '3PA': 164},
     'ORL': {'FG': 163, 'FGA': 338, '3P': 52, '3PA': 144},
     'POR': {'FG': 176, 'FGA': 368, '3P': 67, '3PA': 142},
     'MEM': {'FG': 164, 'FGA': 368, '3P': 43, '3PA': 143},
     'WAS': {'FG': 159, 'FGA': 358, '3P': 35, '3PA': 104},
     'PHO': {'FG': 172, 'FGA': 361, '3P': 49, '3PA': 131},
     'MIL': {'FG': 167, 'FGA': 342, '3P': 54, '3PA': 164},
     'BOS': {'FG': 166, 'FGA': 341, '3P': 59, '3PA': 139},
     'SAS': {'FG': 176, 'FGA': 362, '3P': 46, '3PA': 110},
     'SAC': {'FG': 184, 'FGA': 383, '3P': 52, '3PA': 142},
     'DAL': {'FG': 160, 'FGA': 375, '3P': 53, '3PA': 169},
     'HOU': {'FG': 158, 'FGA': 363, '3P': 80, '3PA': 219},
     'DEN': {'FG': 175, 'FGA': 351, '3P': 43, '3PA': 124},
     'MIA': {'FG': 146, 'FGA': 319, '3P': 63, '3PA': 163},
     'OKC': {'FG': 112, 'FGA': 243, '3P': 30, '3PA': 86},
     'IND': {'FG': 175, 'FGA': 356, '3P': 44, '3PA': 122},
     'PHI': {'FG': 130, 'FGA': 262, '3P': 34, '3PA': 85},
     'TOR': {'FG': 102, 'FGA': 229, '3P': 42, '3PA': 99}}



**In [24]:**

{% highlight python %}
for t in after:
    after[t]['FG%'] = after[t]['FG']/float(after[t]['FGA'])
    after[t]['3P%'] = after[t]['3P']/float(after[t]['3PA'])
after = pd.DataFrame(after).transpose()
after
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
      <th>FG</th>
      <th>FGA</th>
      <th>3P</th>
      <th>3PA</th>
      <th>FG%</th>
      <th>3P%</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>NOP</th>
      <td>162.0</td>
      <td>343.0</td>
      <td>45.0</td>
      <td>124.0</td>
      <td>0.472303</td>
      <td>0.362903</td>
    </tr>
    <tr>
      <th>UTA</th>
      <td>146.0</td>
      <td>338.0</td>
      <td>46.0</td>
      <td>153.0</td>
      <td>0.431953</td>
      <td>0.300654</td>
    </tr>
    <tr>
      <th>LAL</th>
      <td>168.0</td>
      <td>412.0</td>
      <td>37.0</td>
      <td>158.0</td>
      <td>0.407767</td>
      <td>0.234177</td>
    </tr>
    <tr>
      <th>LAC</th>
      <td>161.0</td>
      <td>338.0</td>
      <td>63.0</td>
      <td>143.0</td>
      <td>0.476331</td>
      <td>0.440559</td>
    </tr>
    <tr>
      <th>BRK</th>
      <td>170.0</td>
      <td>379.0</td>
      <td>53.0</td>
      <td>164.0</td>
      <td>0.448549</td>
      <td>0.323171</td>
    </tr>
    <tr>
      <th>ORL</th>
      <td>163.0</td>
      <td>338.0</td>
      <td>52.0</td>
      <td>144.0</td>
      <td>0.482249</td>
      <td>0.361111</td>
    </tr>
    <tr>
      <th>POR</th>
      <td>176.0</td>
      <td>368.0</td>
      <td>67.0</td>
      <td>142.0</td>
      <td>0.478261</td>
      <td>0.471831</td>
    </tr>
    <tr>
      <th>MEM</th>
      <td>164.0</td>
      <td>368.0</td>
      <td>43.0</td>
      <td>143.0</td>
      <td>0.445652</td>
      <td>0.300699</td>
    </tr>
    <tr>
      <th>WAS</th>
      <td>159.0</td>
      <td>358.0</td>
      <td>35.0</td>
      <td>104.0</td>
      <td>0.444134</td>
      <td>0.336538</td>
    </tr>
    <tr>
      <th>PHO</th>
      <td>172.0</td>
      <td>361.0</td>
      <td>49.0</td>
      <td>131.0</td>
      <td>0.476454</td>
      <td>0.374046</td>
    </tr>
    <tr>
      <th>MIL</th>
      <td>167.0</td>
      <td>342.0</td>
      <td>54.0</td>
      <td>164.0</td>
      <td>0.488304</td>
      <td>0.329268</td>
    </tr>
    <tr>
      <th>BOS</th>
      <td>166.0</td>
      <td>341.0</td>
      <td>59.0</td>
      <td>139.0</td>
      <td>0.486804</td>
      <td>0.424460</td>
    </tr>
    <tr>
      <th>SAS</th>
      <td>176.0</td>
      <td>362.0</td>
      <td>46.0</td>
      <td>110.0</td>
      <td>0.486188</td>
      <td>0.418182</td>
    </tr>
    <tr>
      <th>SAC</th>
      <td>184.0</td>
      <td>383.0</td>
      <td>52.0</td>
      <td>142.0</td>
      <td>0.480418</td>
      <td>0.366197</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>160.0</td>
      <td>375.0</td>
      <td>53.0</td>
      <td>169.0</td>
      <td>0.426667</td>
      <td>0.313609</td>
    </tr>
    <tr>
      <th>HOU</th>
      <td>158.0</td>
      <td>363.0</td>
      <td>80.0</td>
      <td>219.0</td>
      <td>0.435262</td>
      <td>0.365297</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>175.0</td>
      <td>351.0</td>
      <td>43.0</td>
      <td>124.0</td>
      <td>0.498575</td>
      <td>0.346774</td>
    </tr>
    <tr>
      <th>MIA</th>
      <td>146.0</td>
      <td>319.0</td>
      <td>63.0</td>
      <td>163.0</td>
      <td>0.457680</td>
      <td>0.386503</td>
    </tr>
    <tr>
      <th>OKC</th>
      <td>112.0</td>
      <td>243.0</td>
      <td>30.0</td>
      <td>86.0</td>
      <td>0.460905</td>
      <td>0.348837</td>
    </tr>
    <tr>
      <th>IND</th>
      <td>175.0</td>
      <td>356.0</td>
      <td>44.0</td>
      <td>122.0</td>
      <td>0.491573</td>
      <td>0.360656</td>
    </tr>
    <tr>
      <th>PHI</th>
      <td>130.0</td>
      <td>262.0</td>
      <td>34.0</td>
      <td>85.0</td>
      <td>0.496183</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>TOR</th>
      <td>102.0</td>
      <td>229.0</td>
      <td>42.0</td>
      <td>99.0</td>
      <td>0.445415</td>
      <td>0.424242</td>
    </tr>
  </tbody>
</table>
</div>


 
Let's define a new dataframe `diff` that contains the difference in FG% and 3P%
for all teams in the bubble. 

**In [28]:**

{% highlight python %}
diff = pd.DataFrame()
diff['FG% after - FG% before'] = (after['FG%']-before['FG%'])*100
diff['3P% after - 3P% before'] = (after['3P%']-before['3P%'])*100
diff.dropna(inplace=True)
diff
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
      <th>FG% after - FG% before</th>
      <th>3P% after - 3P% before</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BOS</th>
      <td>2.751843</td>
      <td>6.190335</td>
    </tr>
    <tr>
      <th>BRK</th>
      <td>0.468277</td>
      <td>-1.690356</td>
    </tr>
    <tr>
      <th>DAL</th>
      <td>-3.575245</td>
      <td>-5.558809</td>
    </tr>
    <tr>
      <th>DEN</th>
      <td>2.799744</td>
      <td>-1.083835</td>
    </tr>
    <tr>
      <th>HOU</th>
      <td>-1.829680</td>
      <td>1.713296</td>
    </tr>
    <tr>
      <th>IND</th>
      <td>1.455354</td>
      <td>-0.196318</td>
    </tr>
    <tr>
      <th>LAC</th>
      <td>1.213899</td>
      <td>7.409689</td>
    </tr>
    <tr>
      <th>LAL</th>
      <td>-7.690768</td>
      <td>-12.069406</td>
    </tr>
    <tr>
      <th>MEM</th>
      <td>-2.432922</td>
      <td>-5.095994</td>
    </tr>
    <tr>
      <th>MIA</th>
      <td>-1.187863</td>
      <td>0.328011</td>
    </tr>
    <tr>
      <th>MIL</th>
      <td>1.090868</td>
      <td>-2.650860</td>
    </tr>
    <tr>
      <th>NOP</th>
      <td>0.991758</td>
      <td>-0.903534</td>
    </tr>
    <tr>
      <th>OKC</th>
      <td>-1.238270</td>
      <td>-0.579538</td>
    </tr>
    <tr>
      <th>ORL</th>
      <td>4.053711</td>
      <td>1.976496</td>
    </tr>
    <tr>
      <th>PHI</th>
      <td>3.117707</td>
      <td>3.840156</td>
    </tr>
    <tr>
      <th>PHO</th>
      <td>1.227493</td>
      <td>2.130441</td>
    </tr>
    <tr>
      <th>POR</th>
      <td>1.760732</td>
      <td>9.974840</td>
    </tr>
    <tr>
      <th>SAC</th>
      <td>2.097272</td>
      <td>0.206920</td>
    </tr>
    <tr>
      <th>SAS</th>
      <td>1.634038</td>
      <td>4.725866</td>
    </tr>
    <tr>
      <th>TOR</th>
      <td>-1.283758</td>
      <td>5.277767</td>
    </tr>
    <tr>
      <th>UTA</th>
      <td>-4.264553</td>
      <td>-8.252822</td>
    </tr>
    <tr>
      <th>WAS</th>
      <td>-1.661150</td>
      <td>-3.568638</td>
    </tr>
  </tbody>
</table>
</div>


 
Now, let's plot the results of FG% after - FG% before and 3P% after - 3P% before
in descending order: 

**In [31]:**

{% highlight python %}
ax = diff.sort_values('FG% after - FG% before', ascending=False).plot.bar(y='FG% after - FG% before', rot=0, figsize=(20, 10))
ax.set_title('Difference in FG% (In bubble - Before bubble)')
ax.set_xlabel('Team')
ax.set_ylabel('Difference in FG%')
ax.get_legend().remove()
{% endhighlight %}

 
![png](https://i.imgur.com/2h3NN4Y.png) 


**In [30]:**

{% highlight python %}
ax = diff.sort_values('3P% after - 3P% before', ascending=False).plot.bar(y='3P% after - 3P% before', rot=0, figsize=(20, 10))
ax.set_title('Difference in 3P% (In bubble - Before bubble)')
ax.set_xlabel('Team')
ax.set_ylabel('Difference in 3P%')
ax.get_legend().remove()
{% endhighlight %}

 
![png](https://i.imgur.com/rFvYZfi.png) 


**In [None]:**

{% highlight python %}

{% endhighlight %}
