---
title: "Plotting the NBA Efficiency Landscape using `Bokeh`"
date: 2019-11-05
permalink: /projects/2019/11/05/plotting-efficiency-landscape
tags:
    - python
    - notebook
--- 

## Introduction

Kirk Goldsberry often likes to post his definition of the [NBA Efficiency Landscape](https://twitter.com/kirkgoldsberry/status/1190020220376633344?lang=en). Why don't we try and build an engine that can create this dynamically at any time?
 
## Data Aggregation

First, let's obtain all the data from [basketball
reference](https://www.basketball-reference.com/leagues/NBA_2020_ratings.html)

The data required includes:
- Team name
- Offensive rating
- Defensive rating
- Net rating 

**In [1]:**

{% highlight python %}
from bs4 import BeautifulSoup
import requests

r = requests.get('https://www.basketball-reference.com/leagues/NBA_2020_ratings.html')
team_names = []
off_rtgs = []
def_rtgs = []
net_rtgs = []
d = {}
if r.status_code==200:
    soup = BeautifulSoup(r.content, 'html.parser')
    table = soup.find('tbody')
    for row in table.find_all('tr'):
        for td in row.find_all('td'):
            stat = td['data-stat']
            if stat=='team_name':
                a = td.find('a')
                team_name = a.contents[0]
                team_names.append(team_name)
            elif stat=='off_rtg':
                off_rtg = float(td.contents[0])
                off_rtgs.append(off_rtg)
            elif stat=='def_rtg':
                def_rtg = float(td.contents[0])
                def_rtgs.append(def_rtg)
            elif stat=='net_rtg':
                net_rtg = float(td.contents[0])
                net_rtgs.append(net_rtg)
                
print(team_names)
print(off_rtgs)
print(def_rtgs)
print(net_rtgs)
{% endhighlight %}

    ['Milwaukee Bucks', 'Los Angeles Lakers', 'Dallas Mavericks', 'Boston Celtics', 'Los Angeles Clippers', 'Toronto Raptors', 'Utah Jazz', 'Houston Rockets', 'Philadelphia 76ers', 'Denver Nuggets', 'Oklahoma City Thunder', 'Miami Heat', 'Indiana Pacers', 'Orlando Magic', 'San Antonio Spurs', 'New Orleans Pelicans', 'Phoenix Suns', 'Portland Trail Blazers', 'Brooklyn Nets', 'Memphis Grizzlies', 'Detroit Pistons', 'Chicago Bulls', 'Sacramento Kings', 'Minnesota Timberwolves', 'Washington Wizards', 'New York Knicks', 'Charlotte Hornets', 'Golden State Warriors', 'Atlanta Hawks', 'Cleveland Cavaliers']
    [114.17, 113.37, 118.24, 113.67, 113.11, 111.59, 112.99, 114.79, 109.85, 112.47, 111.75, 112.8, 111.61, 106.48, 112.12, 110.38, 110.5, 111.42, 106.69, 110.41, 110.76, 106.28, 108.67, 107.62, 111.91, 105.65, 106.88, 105.05, 104.56, 106.71]
    [102.1, 106.29, 111.32, 106.43, 107.12, 105.63, 107.36, 110.36, 106.25, 108.93, 109.22, 109.49, 108.27, 107.1, 113.08, 113.5, 111.64, 113.79, 109.02, 112.88, 112.46, 108.3, 111.85, 111.22, 116.7, 113.57, 114.34, 113.69, 113.73, 116.15]
    [12.08, 7.09, 6.92, 7.25, 5.99, 5.96, 5.63, 4.44, 3.6, 3.54, 2.53, 3.31, 3.34, -0.63, -0.96, -3.12, -1.14, -2.36, -2.33, -2.47, -1.7, -2.02, -3.17, -3.6, -4.79, -7.91, -7.46, -8.64, -9.17, -9.44]

 
Using a `json` file that encodes team names, abbreviations, and colors, let's
get each team's logo to plot. 

**In [2]:**

{% highlight python %}
import json
logos = []
with open('teams.json', 'r') as f:
    teams = json.load(f)
    for team in team_names:
        for t in teams:
            if t['teamName']==team:
                abbr = t['abbreviation'].lower()
                url = f"http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/{t['abbreviation'].lower()}.png"
                logos.append(url)
with open('colors.json', 'r') as f:
    colors = json.load(f)
logos
{% endhighlight %}




    ['http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/mil.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/lal.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/dal.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/bos.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/lac.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/tor.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/uta.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/hou.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/phi.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/den.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/okc.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/mia.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/ind.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/orl.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/sas.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/nop.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/phx.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/por.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/bkn.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/mem.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/det.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/chi.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/sac.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/min.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/was.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/nyk.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/cha.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/gsw.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/atl.png',
     'http://i.cdn.turner.com/nba/nba/.element/img/1.0/teamsites/logos/teamlogos_500x500/cle.png']


 
## Plotting Using Bokeh 

**In [3]:**

{% highlight python %}
from bokeh.plotting import figure, show, output_notebook
from bokeh.models import Label, Slope, Title
from datetime import datetime
output_notebook()

min_x = min(off_rtgs)
max_x = max(off_rtgs)
min_y = min(def_rtgs)
max_y = max(def_rtgs)

avg_x = sum(off_rtgs)/float(len(off_rtgs))
avg_y = sum(def_rtgs)/float(len(def_rtgs))

# Create figure
p = figure(plot_width=1000, plot_height=1000, 
           x_range=(min_x-1,max_x+1), y_range=(max_y+1, min_y-1,),  
           background_fill_color="beige", border_fill_color="beige")

# Create titles
now = datetime.now()
date_str = datetime.strftime(now, "%B %d, %Y")
p.add_layout(Title(text=f'AS OF {date_str.upper()}', text_font_style="italic", text_font='monospace'), 'above')
p.add_layout(Title(text="NBA EFFICIENCY LANDSCAPE", text_font_size="16pt", text_font='monospace', render_mode='canvas'), 'above')

# Center x and y axes
p.xaxis.fixed_location = avg_x
p.yaxis.fixed_location = avg_y

# Add axes labels
x_axis_label = Label(x=min_x, y=avg_y+0.2,
                 text='OFFENSIVE EFFICIENCY', render_mode='canvas', text_font='monospace')
y_axis_label = Label(x=avg_x+0.1, y=min_y,
                 text='DEFENSIVE EFFICIENCY', angle=-90, angle_units='deg', render_mode='canvas', text_font='monospace')
p.add_layout(x_axis_label)
p.add_layout(y_axis_label)

# Add slope to indicate y=-x line
y_int = -1*avg_x+avg_y
slope = Slope(gradient=1, y_intercept=y_int,
              line_color='black', line_dash='dashed', line_width=3.5)

p.add_layout(slope)

# Add labels to indicate + and - teams
y_val = 1*min_x+y_int
pos_teams_label = Label(x=min_x+0.1, y=y_val-0.1,
                 text='POSITIVE TEAMS', angle=-42, angle_units='deg', render_mode='canvas', text_font='monospace')
neg_teams_label = Label(x=min_x-0.4, y=y_val+0.4,
                 text='NEGATIVE TEAMS', angle=-42, angle_units='deg', render_mode='canvas', text_font='monospace')
p.add_layout(pos_teams_label)
p.add_layout(neg_teams_label)

# Add images
for i in range(0, len(logos)):
    p.image_url(url=[logos[i]],
             x=off_rtgs[i], y=def_rtgs[i], w=2, h=2, anchor="center")

extremes = []
for el in sorted(off_rtgs)[:3]+sorted(off_rtgs)[-3:]:
    i = off_rtgs.index(el)
    extremes.append(team_names[i])

for el in sorted(def_rtgs)[:3]+sorted(def_rtgs)[-3:]:
    i = def_rtgs.index(el)
    extremes.append(team_names[i])  


# Add labels
for team in extremes:
    i = team_names.index(team)
    if off_rtgs[i]>avg_x and def_rtgs[i]>avg_y:
        x_offset = -2.75
        y_offset = -0.25
    elif off_rtgs[i]>avg_x and def_rtgs[i]<avg_y:
        x_offset = -2.75
        y_offset = 2.5
    elif off_rtgs[i]<avg_x and def_rtgs[i]>avg_y:
        x_offset = 0.75
        y_offset = -0.25
    elif off_rtgs[i]<avg_x and def_rtgs[i]<avg_y:
        x_offset=0.75
        y_offset=2.5
    for el in teams:
        if team==el['teamName']:
            abbr = el['abbreviation']
            color = colors[abbr]['main_color']
    
    label = Label(x=off_rtgs[i]+x_offset, y=def_rtgs[i]+y_offset-1.5, 
                  text_font='monospace',
                  text_font_style='bold', 
                  text='#' + str(sorted(net_rtgs)[::-1].index(net_rtgs[i])+1) + ' NET RTG', 
                  render_mode='canvas')
    p.add_layout(label)
    label = Label(x=off_rtgs[i]+x_offset, y=def_rtgs[i]+y_offset-1.0,
                  text='#' + str(sorted(off_rtgs)[::-1].index(off_rtgs[i])+1) + ' OFF RTG', render_mode='canvas', 
                  text_font_style='bold', text_font='monospace')
    p.add_layout(label)
    label = Label(x=off_rtgs[i]+x_offset, y=def_rtgs[i]+y_offset-0.5,
                text='#' + str(sorted(def_rtgs).index(def_rtgs[i])+1) + ' DEF RTG', render_mode='canvas', 
                text_font_style='bold', text_font='monospace')
    p.add_layout(label)
{% endhighlight %}



    <div class="bk-root">
        <a href="https://bokeh.pydata.org" target="_blank" class="bk-logo bk-logo-small bk-logo-notebook"></a>
        <span id="1001">Loading BokehJS ...</span>
    </div>




**In [4]:**

{% highlight python %}
show(p)
{% endhighlight %}








  <div class="bk-root" id="ecb0f1c8-59b8-4366-9e49-213e37c561e3" data-root-id="1002"></div>





**In [None]:**

![img](/assets/2019-11-05-plotting-efficiency-landscape_1.png)
