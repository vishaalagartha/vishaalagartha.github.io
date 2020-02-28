---
title: "Timeline of Player Development: Guards vs. Wings"
date: 2020-02-27
permalink: /notes/2020/02/27/player-development
tags:
    - python    
    - notebook
--- 
# Which advanced positions take the most time to develop?

Recently, Sam Esfandiara [posited a
theory](https://twitter.com/samesfandiari/status/1231947582051454982):

"Theory: In the modern NBA, wings take longest to develop."

This tweet resulted in much controversy and debate. In short, several, including
Sam and Ethan Sherwood Strauss argued in favor of the wing being the position
that takes longest to develop, but others such as Kevin Pelton and Mo Dakhil
argued for the gaurd/point gaurd. 
 
## The Argument for Wings

Sam later posted [this
tweet](https://twitter.com/samesfandiari/status/1231948399869431813). In short,
the argument for wings is that they "have the most responsibility, to cover the
most roles". Sam stated that their roles are:
* Ability to guard multiple positions
* Ability to help
* Play on and off the ball
* ... 
 
## The Argument for Gaurds

Pelton [later rebutted](https://twitter.com/kpelton/status/1232115274431651841).
In short, point guards skills require more experience/skill. 
 
We can investigate this using the [basketball player grades provided by the
BBall-Index](https://www.bball-index.com/2017-18-player-grades/). Having scraped
all the data from 2014-15 to 2017-18 (all available seasons), we can dive into
some meta-statistics to understand how certain positions improved at relevant
skills over the 4 year span. 

**In [2]:**

{% highlight python %}
import pandas as pd
df = pd.read_csv('2017_18.csv')
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
      <th>NAME</th>
      <th>TEAM</th>
      <th>POSITION</th>
      <th>ADVANCED_POSITION</th>
      <th>OFFENSIVE_ROLE</th>
      <th>NUMDATA</th>
      <th>PERIMETER_SHOT</th>
      <th>PERIMETER_SHOT_GRADE</th>
      <th>OFF_BALL_MOVEMENT</th>
      <th>OFF_BALL_MOVEMENT_GRADE</th>
      <th>...</th>
      <th>INTERIOR_DEFENSE</th>
      <th>INTERIOR_DEFENSE_GRADE</th>
      <th>OREB</th>
      <th>OREB_GRADE</th>
      <th>DREB</th>
      <th>DREB_GRADE</th>
      <th>USAGE</th>
      <th>USAGE_GRADE</th>
      <th>SELF_CREATION</th>
      <th>SELF_CREATION_GRADE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>LeBron James</td>
      <td>CLE</td>
      <td>PF</td>
      <td>Wing</td>
      <td>Point Forwards</td>
      <td>3948</td>
      <td>0.756</td>
      <td>B+</td>
      <td>0.958</td>
      <td>A</td>
      <td>...</td>
      <td>0.735</td>
      <td>B</td>
      <td>0.597</td>
      <td>C+</td>
      <td>0.926</td>
      <td>A</td>
      <td>0.993</td>
      <td>High</td>
      <td>0.774</td>
      <td>High</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Klay Thompson</td>
      <td>GSW</td>
      <td>SG</td>
      <td>Wing</td>
      <td>Off-Ball Workers</td>
      <td>3300</td>
      <td>0.978</td>
      <td>A</td>
      <td>0.843</td>
      <td>A-</td>
      <td>...</td>
      <td>0.695</td>
      <td>B</td>
      <td>0.321</td>
      <td>D</td>
      <td>0.501</td>
      <td>C</td>
      <td>0.840</td>
      <td>Medium</td>
      <td>0.196</td>
      <td>Low</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jrue Holiday</td>
      <td>NOP</td>
      <td>SG</td>
      <td>Wing</td>
      <td>Creators</td>
      <td>3275</td>
      <td>0.526</td>
      <td>C</td>
      <td>0.533</td>
      <td>C</td>
      <td>...</td>
      <td>0.564</td>
      <td>C+</td>
      <td>0.532</td>
      <td>C</td>
      <td>0.618</td>
      <td>B-</td>
      <td>0.936</td>
      <td>High</td>
      <td>0.592</td>
      <td>High</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Khris Middleton</td>
      <td>MIL</td>
      <td>SF</td>
      <td>Wing</td>
      <td>Secondary Creators</td>
      <td>3257</td>
      <td>0.741</td>
      <td>B+</td>
      <td>0.357</td>
      <td>D+</td>
      <td>...</td>
      <td>0.500</td>
      <td>C</td>
      <td>0.245</td>
      <td>D-</td>
      <td>0.740</td>
      <td>B+</td>
      <td>0.917</td>
      <td>High</td>
      <td>0.487</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bradley Beal</td>
      <td>WAS</td>
      <td>SG</td>
      <td>Guard</td>
      <td>Secondary Creators</td>
      <td>3193</td>
      <td>0.804</td>
      <td>A-</td>
      <td>0.639</td>
      <td>B-</td>
      <td>...</td>
      <td>0.432</td>
      <td>C-</td>
      <td>0.288</td>
      <td>D</td>
      <td>0.375</td>
      <td>D+</td>
      <td>0.952</td>
      <td>High</td>
      <td>0.519</td>
      <td>High</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 32 columns</p>
</div>


 
Let's classify each skill as either a *wing skill* or a *guard skill*. 

**In [3]:**

{% highlight python %}
df.columns
{% endhighlight %}




    Index(['NAME', 'TEAM', 'POSITION', 'ADVANCED_POSITION', 'OFFENSIVE_ROLE',
           'NUMDATA', 'PERIMETER_SHOT', 'PERIMETER_SHOT_GRADE',
           'OFF_BALL_MOVEMENT', 'OFF_BALL_MOVEMENT_GRADE', 'ONE_ON_ONE',
           'ONE_ON_ONE_GRADE', 'FINISHING', 'FINISHING_GRADE', 'ROLL_GRAVITY',
           'ROLL_GRAVITY_GRADE', 'PLAYMAKING', 'PLAYMAKING_GRADE', 'POST_PLAY',
           'POST_PLAY_GRADE', 'PERIMETER_DEFENSE', 'PERIMETER_DEFENSE_GRADE',
           'INTERIOR_DEFENSE', 'INTERIOR_DEFENSE_GRADE', 'OREB', 'OREB_GRADE',
           'DREB', 'DREB_GRADE', 'USAGE', 'USAGE_GRADE', 'SELF_CREATION',
           'SELF_CREATION_GRADE'],
          dtype='object')


 
We can classify these skills into the following categories based on what skills
**epitomize** each position:

## Wing Skills

* `PERIMETER_SHOT`
* `OFF_BALL_MOVEMENT`
* `ONE_ON_ONE`
* `FINISHING`
* `PERIMETER_DEFENSE`
* `SELF_CREATION`

## Guard Skills

* `PERIMETER_SHOT`
* `ONE_ON_ONE`
* `FINISHING`
* `PLAYMAKING`
* `SELF_CREATION` 

**In [4]:**

{% highlight python %}
def get_wing_data(year):
    wing_skills = ['PERIMETER_SHOT', 'OFF_BALL_MOVEMENT', 
                   'ONE_ON_ONE', 'FINISHING', 'PERIMETER_DEFENSE', 'SELF_CREATION']
    y_df = pd.read_csv(f'{year}.csv')
    wing_y_df = y_df[y_df['ADVANCED_POSITION']=='Wing']
    df2 = wing_y_df[['NAME']+wing_skills]
    df2['YEAR'] = year.replace('_','-')
    return df2

def get_guard_data(year):
    guard_skills = ['PERIMETER_SHOT',
                   'ONE_ON_ONE', 'FINISHING', 'PLAYMAKING', 'SELF_CREATION']
    y_df = pd.read_csv(f'{year}.csv')
    guard_y_df = y_df[y_df['ADVANCED_POSITION']=='Guard']
    df2 = guard_y_df[['NAME']+guard_skills]
    df2['YEAR'] = year.replace('_','-')
    return df2
{% endhighlight %}

**In [5]:**

{% highlight python %}
wing_df = pd.DataFrame()
guard_df = pd.DataFrame()
for y in range(2014, 2018):
    s = f'{y}_{str(y+1)[2:]}'
    wing_df = wing_df.append(get_wing_data(s))
    guard_df = guard_df.append(get_guard_data(s))
wing_df = wing_df.reset_index().drop('index', axis=1)
guard_df = guard_df.reset_index().drop('index', axis=1)
{% endhighlight %}

    /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/ipykernel_launcher.py:7: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      import sys
    /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/ipykernel_launcher.py:16: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      app.launch_new_instance()


**In [6]:**

{% highlight python %}
wing_df.head()
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
      <th>NAME</th>
      <th>PERIMETER_SHOT</th>
      <th>OFF_BALL_MOVEMENT</th>
      <th>ONE_ON_ONE</th>
      <th>FINISHING</th>
      <th>PERIMETER_DEFENSE</th>
      <th>SELF_CREATION</th>
      <th>YEAR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>James Harden</td>
      <td>0.911</td>
      <td>0.901</td>
      <td>0.986</td>
      <td>0.778</td>
      <td>0.774</td>
      <td>0.764</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Trevor Ariza</td>
      <td>0.667</td>
      <td>0.339</td>
      <td>0.444</td>
      <td>0.608</td>
      <td>0.745</td>
      <td>0.190</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>2</th>
      <td>LeBron James</td>
      <td>0.803</td>
      <td>0.913</td>
      <td>0.880</td>
      <td>0.992</td>
      <td>0.554</td>
      <td>0.816</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Klay Thompson</td>
      <td>0.980</td>
      <td>0.700</td>
      <td>0.972</td>
      <td>0.742</td>
      <td>0.604</td>
      <td>0.321</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Joe Johnson</td>
      <td>0.732</td>
      <td>0.475</td>
      <td>0.890</td>
      <td>0.239</td>
      <td>0.365</td>
      <td>0.583</td>
      <td>2014-15</td>
    </tr>
  </tbody>
</table>
</div>



**In [7]:**

{% highlight python %}
guard_df.head()
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
      <th>NAME</th>
      <th>PERIMETER_SHOT</th>
      <th>ONE_ON_ONE</th>
      <th>FINISHING</th>
      <th>PLAYMAKING</th>
      <th>SELF_CREATION</th>
      <th>YEAR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Stephen Curry</td>
      <td>0.982</td>
      <td>0.955</td>
      <td>0.910</td>
      <td>0.964</td>
      <td>0.570</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Chris Paul</td>
      <td>0.929</td>
      <td>0.738</td>
      <td>0.480</td>
      <td>0.971</td>
      <td>0.802</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Kyrie Irving</td>
      <td>0.965</td>
      <td>0.990</td>
      <td>0.642</td>
      <td>0.877</td>
      <td>0.734</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Damian Lillard</td>
      <td>0.734</td>
      <td>0.970</td>
      <td>0.836</td>
      <td>0.959</td>
      <td>0.718</td>
      <td>2014-15</td>
    </tr>
    <tr>
      <th>4</th>
      <td>John Wall</td>
      <td>0.514</td>
      <td>0.653</td>
      <td>0.683</td>
      <td>0.970</td>
      <td>0.768</td>
      <td>2014-15</td>
    </tr>
  </tbody>
</table>
</div>


 
Now, let's add another column to the current dataframes indicating the years of
experience the player has in the league. 

**In [49]:**

{% highlight python %}
from basketball_reference_scraper.players import get_stats

for i, r in list(guard_df.iterrows()):
    print(r['NAME'])
    try:
        stats_df = get_stats(r['NAME'])
        rookie_year = int(stats_df.iloc[0]['SEASON'][:4])
        years_in_league = int(r['YEAR'][:4])-rookie_year
        guard_df.at[i, 'YRS_IN_LEAGUE'] = years_in_league
    except:
        pass
for i, r in list(wing_df.iterrows()):
    print(r['NAME'])
    try:
        stats_df = get_stats(r['NAME'])
        rookie_year = int(stats_df.iloc[0]['SEASON'][:4])
        years_in_league = int(r['YEAR'][:4])-rookie_year
        wing_df.at[i, 'YRS_IN_LEAGUE'] = years_in_league
    except:
        pass
{% endhighlight %}

    Stephen Curry
    Chris Paul
    Kyrie Irving
    Damian Lillard
    John Wall
    Eric Bledsoe
    Jeff Teague
    Ty Lawson
    Goran Dragic
    Victor Oladipo
    Avery Bradley
    Kyle Lowry
    Bradley Beal
    Elfrid Payton
    Mike Conley
    Jarrett Jack
    Mario Chalmers
    Michael Carter-Williams
    Deron Williams
    Russell Westbrook
    Trey Burke
    Reggie Jackson
    Iman Shumpert
    Tony Parker
    Eric Gordon
    Jason Terry
    Kemba Walker
    Louis Williams
    Greivis Vasquez
    Rajon Rondo
    Brandon Knight
    Aaron Brooks
    Derrick Rose
    Maurice Williams
    DJ Augustin
    Norris Cole
    Tony Allen
    Jeremy Lin
    Zach LaVine
    Marcus Smart
    Matthew Dellavedova
    Rodney Stuckey
    Shane Larkin
    Isaiah Thomas
    Shaun Livingston
    Jerryd Bayless
    Dante Exum
    Austin Rivers
    Dennis Schroder
    Devin Harris
    Kirk Hinrich
    Patrick Beverley
    Wayne Ellington
    Beno Udrih
    Steve Blake
    Darren Collison
    J.J. Barea
    Marco Belinelli
    Pablo Prigioni
    Jordan Clarkson
    Cory Joseph
    Langston Galloway
    Ray McCallum
    CJ Watson
    Jameer Nelson
    Ramon Sessions
    Jrue Holiday
    Brian Roberts
    Jose Calderon
    George Hill
    Andre Miller
    Leandro Barbosa
    Gary Neal
    Brandon Jennings
    CJ McCollum
    Donald Sloan
    Shabazz Napier
    Ronnie Price
    Willie Green
    Isaiah Canaan
    Shelvin Mack
    Patty Mills
    Tony Wroten
    Ish Smith
    Markel Brown
    Ben Gordon
    Alexey Shved
    Garrett Temple
    Marcus Thornton
    Ricky Rubio
    Luke Ridnour
    Phil Pressey
    Tyler Johnson
    Nate Robinson
    Lorenzo Brown
    Archie Goodwin
    Jordan Farmar
    ETwaun Moore
    Jimmer Fredette
    Spencer Dinwiddie
    Tyler Ennis
    Kendall Marshall
    Jason Richardson
    Erick Green
    Troy Daniels
    Sebastian Telfair
    Raymond Felton
    Nick Johnson
    Darius Morris
    John Lucas III
    Jordan Adams
    Nate Wolters
    Tim Frazier
    Larry Drew
    Ian Clark
    Jorge Gutierrez
    Toney Douglas
    Bryce Cotton
    Dwight Buycks
    CJ Wilcox
    Will Bynum
    Jared Cunningham
    Vander Blue
    Jannero Pargo
    Will Cherry
    Russ Smith
    Gal Mekel
    Jerel McNeal
    David Stockton
    Toure Murry
    Seth Curry
    Kalin Lucas
    Malcolm Lee
    Kyle Lowry
    Russell Westbrook
    Stephen Curry
    CJ McCollum
    Kemba Walker
    Damian Lillard
    Monta Ellis
    Isaiah Thomas
    Goran Dragic
    Giannis Antetokounmpo
    John Wall
    George Hill
    Dwyane Wade
    Reggie Jackson
    Jordan Clarkson
    Chris Paul
    Rajon Rondo
    Jeff Teague
    Cory Joseph
    Kyrie Irving
    Victor Oladipo
    Raymond Felton
    Will Barton
    Ricky Rubio
    Zach LaVine
    JJ Redick
    DAngelo Russell
    Tony Parker
    Ish Smith
    Jeremy Lin
    Darren Collison
    Patrick Beverley
    Deron Williams
    Elfrid Payton
    Matthew Dellavedova
    Derrick Rose
    Emmanuel Mudiay
    Shaun Livingston
    Langston Galloway
    Jose Calderon
    Isaiah Canaan
    Louis Williams
    Brandon Knight
    Marcus Smart
    Jrue Holiday
    Patty Mills
    Dennis Schroder
    J.J. Barea
    Mike Conley
    Shane Larkin
    Tony Allen
    Ramon Sessions
    Michael Carter-Williams
    Austin Rivers
    T.J. McConnell
    Raul Neto
    Josh Richardson
    Ty Lawson
    Eric Gordon
    Rodney Stuckey
    Jason Terry
    Mario Chalmers
    Trey Burke
    Donald Sloan
    Jerian Grant
    ETwaun Moore
    Toney Douglas
    Ronnie Price
    Norris Cole
    DJ Augustin
    Archie Goodwin
    Aaron Brooks
    Eric Bledsoe
    Shelvin Mack
    Jameer Nelson
    Steve Blake
    Jarrett Jack
    Markel Brown
    Tyler Johnson
    Marcelo Huertas
    Brandon Jennings
    Pablo Prigioni
    Jonathan Simmons
    Maurice Williams
    Alec Burks
    Justin Holiday
    Cameron Payne
    Tim Frazier
    Ian Clark
    Beno Udrih
    Seth Curry
    Kirk Hinrich
    CJ Watson
    Tyler Ennis
    Shabazz Napier
    Tyus Jones
    Brian Roberts
    Andre Miller
    Ray McCallum
    Greivis Vasquez
    Terry Rozier III
    Jared Cunningham
    Jordan Farmar
    Kendall Marshall
    Joe Young
    Xavier Munford
    Phil Pressey
    Delon Wright
    Jordan McRae
    Briante Weber
    Spencer Dinwiddie
    Tony Wroten
    Orlando Johnson
    Dahntay Jones
    Jorge Gutierrez
    Russ Smith
    Lorenzo Brown
    Andrew Goudelock
    Elliot Williams
    Erick Green
    Bryce Cotton
    Keith Appling
    Nate Robinson
    James Harden
    John Wall
    Stephen Curry
    Kyrie Irving
    Isaiah Thomas
    Russell Westbrook
    CJ McCollum
    Damian Lillard
    Jeff Teague
    Kemba Walker
    Devin Booker
    Dennis Schroder
    Kyle Lowry
    Mike Conley
    Avery Bradley
    Ricky Rubio
    Goran Dragic
    Elfrid Payton
    Jordan Clarkson
    Patrick Beverley
    Tim Hardaway Jr.
    Jamal Crawford
    Louis Williams
    Cory Joseph
    Iman Shumpert
    Jrue Holiday
    Nik Stauskas
    Chris Paul
    Tyler Johnson
    Eric Bledsoe
    Patty Mills
    Malcolm Brogdon
    Matthew Dellavedova
    T.J. McConnell
    Derrick Rose
    Darren Collison
    Jameer Nelson
    Seth Curry
    Dwyane Wade
    Ish Smith
    Rajon Rondo
    Raymond Felton
    George Hill
    DAngelo Russell
    Tony Parker
    Ty Lawson
    Garrett Temple
    Will Barton
    Isaiah Whitehead
    Justin Holiday
    Josh Richardson
    Andrew Harrison
    Manu Ginobili
    Shaun Livingston
    Terry Rozier III
    DJ Augustin
    Tim Frazier
    Sergio Rodriguez
    Langston Galloway
    Jason Terry
    Reggie Jackson
    Emmanuel Mudiay
    Shelvin Mack
    Spencer Dinwiddie
    Kris Dunn
    Dante Exum
    Randy Foye
    Patrick McCaw
    Malcom Delaney
    Yogi Ferrell
    Joe Harris
    Tyler Ulis
    Devin Harris
    Jerian Grant
    CJ Watson
    Denzel Valentine
    Semaj Christon
    Leandro Barbosa
    Aaron Brooks
    Michael Carter-Williams
    Jeremy Lin
    Ramon Sessions
    DeAndre Liggins
    Tyus Jones
    J.J. Barea
    Tomas Satoransky
    Trey Burke
    Rodney Stuckey
    Alec Burks
    Jose Calderon
    Tyler Ennis
    Beno Udrih
    Shabazz Napier
    Delon Wright
    Dejounte Murray
    Rashad Vaughn
    Cameron Payne
    Wade Baldwin
    Brian Roberts
    Raul Neto
    Toney Douglas
    Kay Felder
    Jordan McRae
    Darrun Hilliard
    Bryn Forbes
    Fred VanVleet
    Sheldon McClellan
    Marcelo Huertas
    Jonathan Gibson
    Briante Weber
    Quinn Cook
    Nicolas Laprovittola
    Malik Beasley
    Norris Cole
    Bobby Brown
    Joe Young
    Ronnie Price
    CJ Wilcox
    Gary Payton II
    Tim Quarterman
    Pierre Jackson
    Jerryd Bayless
    Isaiah Taylor
    Marcus Georges-Hunt
    Lamar Patterson
    Greivis Vasquez
    Jordan Farmar
    Jarrett Jack
    Michael Gbinije
    Manny Harris
    Gary Neal
    Demetrius Jackson
    John Lucas III
    Bradley Beal
    James Harden
    Russell Westbrook
    Ben Simmons
    Donovan Mitchell
    Kyle Lowry
    E'Twaun Moore
    Damian Lillard
    Terry Rozier
    Kemba Walker
    Will Barton
    Lou Williams
    Jamal Murray
    Eric Bledsoe
    Goran Dragic
    Jeff Teague
    Ricky Rubio
    George Hill
    Chris Paul
    Cory Joseph
    Spencer Dinwiddie
    Patty Mills
    Darren Collison
    Stephen Curry
    Jordan Clarkson
    Wayne Ellington
    Tyler Johnson
    Dennis Schroder
    Dennis Smith
    Ish Smith
    De'Aaron Fox
    Rajon Rondo
    Joe Harris
    Kyrie Irving
    Devin Booker
    T.J. McConnell
    Dejounte Murray
    Elfrid Payton
    Lonzo Ball
    D.J. Augustin
    Frank Ntilikina
    Tomas Satoransky
    Jerian Grant
    Dwyane Wade
    Tyler Ulis
    Delon Wright
    Ian Clark
    John Wall
    Fred VanVleet
    Malcolm Brogdon
    Tyreke Evans
    J.J. Barea
    Shabazz Napier
    Jarrett Jack
    Kris Dunn
    Tyus Jones
    Shaun Livingston
    Raymond Felton
    Avery Bradley
    Mario Chalmers
    Shelvin Mack
    Devin Harris
    Andrew Harrison
    Emmanuel Mudiay
    D'Angelo Russell
    Reggie Jackson
    Alec Burks
    Isaiah Taylor
    Tony Parker
    Milos Teodosic
    Jose Calderon
    Malcolm Delaney
    Jameer Nelson
    Frank Mason
    Patrick McCaw
    Shane Larkin
    Quinn Cook
    Isaiah Thomas
    Jason Terry
    Tyrone Wallace
    Tim Frazier
    Michael Carter-Williams
    Matthew Dellavedova
    Trey Burke
    Jawun Evans
    Terrance Ferguson
    Dwayne Bacon
    Mike James
    Tyler Ennis
    Kobi Simmons
    Cameron Payne
    Raul Neto
    Alex Caruso
    Joe Young
    Derrick Rose
    Nik Stauskas
    Dwight Buycks
    Isaiah Canaan
    Damion Lee
    Ramon Sessions
    Shaquille Harrison
    Mike Conley
    Dante Exum
    Rodney McGruder
    Ryan Arcidiacono
    Rodney Purvis
    Markelle Fultz
    Iman Shumpert
    Briante Weber
    Aaron Harrison
    Gary Payton
    Josh Magette
    Brandon Jennings
    Andrew White
    Aaron Brooks
    Isaiah Whitehead
    Julyan Stone
    Lorenzo Brown
    Derrick White
    Derrick Walton
    Kay Felder
    Milton Doyle
    Xavier Rathan-Mayes
    Bobby Brown
    Kadeem Allen
    Wade Baldwin
    Josh Gray
    Furkan Korkmaz
    Demetrius Jackson
    Jordan Crawford
    London Perrantes
    Gian Clavell
    Walt Lemon Jr.
    Markel Brown
    Marcus Paige
    Monte Morris
    Jeremy Lin
    Xavier Munford
    Nate Wolters
    Scotty Hopson
    Reggie Hearn
    Jacob Pullen
    PJ Dozier
    James Harden
    Trevor Ariza
    LeBron James
    Klay Thompson
    Joe Johnson
    Jimmy Butler
    Harrison Barnes
    Andrew Wiggins
    JJ Redick
    Kyle Korver
    Monta Ellis
    Tyreke Evans
    Jeff Green
    DeMarre Carroll
    Giannis Antetokounmpo
    Courtney Lee
    Andre Iguodala
    Josh Smith
    Matt Barnes
    Ben McLemore
    JR Smith
    Thaddeus Young
    Gordon Hayward
    Khris Middleton
    Nicolas Batum
    Kentavious Caldwell-Pope
    Arron Afflalo
    Danny Green
    Corey Brewer
    Wilson Chandler
    Luol Deng
    Rudy Gay
    PJ Tucker
    Solomon Hill
    Evan Turner
    Tobias Harris
    Gerald Henderson
    Kawhi Leonard
    Patrick Patterson
    DeMar DeRozan
    Wesley Johnson
    Mike Dunleavy
    Chandler Parsons
    Paul Pierce
    Dion Waiters
    Terrence Ross
    Boris Diaw
    Greg Monroe
    Jamal Crawford
    Bojan Bogdanovic
    Marcus Morris
    Wesley Matthews
    Dwyane Wade
    Taj Gibson
    Robert Covington
    Quincy Pondexter
    Luc Mbah a Moute
    Alan Anderson
    OJ Mayo
    CJ Miles
    Jared Dudley
    Anthony Morrow
    Hollis Thompson
    Ryan Anderson
    Otto Porter Jr.
    Jae Crowder
    Kyle Singler
    Dante Cunningham
    Manu Ginobili
    Carlos Boozer
    Joe Ingles
    Evan Fournier
    Kent Bazemore
    Caron Butler
    Michael Kidd-Gilchrist
    Lance Stephenson
    Tony Snell
    Al Farouq Aminu
    Rasual Butler
    Lance Thomas
    Derrick Williams
    Jodie Meeks
    Brandan Wright
    Gerald Green
    Anthony Tolliver
    Carmelo Anthony
    Danilo Gallinari
    Omri Casspi
    Tayshaun Prince
    Jerami Grant
    Kevin Martin
    Mike Scott
    Richard Jefferson
    Quincy Acy
    Vince Carter
    Andre Roberson
    Chase Budinger
    Ryan Kelly
    Kobe Bryant
    JaKarr Sampson
    Nik Stauskas
    Shawn Marion
    Randy Foye
    Shawne Williams
    Rodney Hood
    James Ennis
    Damjan Rudez
    Nick Young
    Will Barton
    James Jones
    Thabo Sefolosha
    Nick Calathes
    Elijah Millsap
    Kevin Durant
    Alec Burks
    Joey Dorsey
    Shabazz Muhammad
    Luke Babbitt
    Chris Copeland
    Kostas Papanikolaou
    Aaron Gordon
    Mike Miller
    Hedo Turkoglu
    Robbie Hummel
    Jabari Parker
    Allen Crabbe
    Gary Harris
    Maurice Harkless
    Travis Wear
    Justin Holiday
    Alonzo Gee
    Cleanthony Early
    Jeremy Lamb
    Perry Jones
    Henry Walker
    TJ Warren
    Danny Granger
    Dorell Wright
    Jabari Brown
    Sergey Karasev
    Cory Jefferson
    Chris Johnson
    Joe Harris
    Jeff Taylor
    Kyle Anderson
    Martell Webster
    Austin Daye
    Reggie Bullock
    James Young
    Doug McDermott
    AJ Price
    John Jenkins
    Gerald Wallace
    Brandon Rush
    John Salmons
    Jeremy Evans
    Glenn Robinson III
    Glenn Robinson
    Ricky Ledo
    Luigi Datome
    Landry Fields
    Devyn Marble
    Francisco Garcia
    Cartier Martin
    Steve Novak
    JaMychal Green
    Dahntay Jones
    Jordan Hamilton
    Quincy Miller
    Elliot Williams
    Reggie Williams
    Chris Douglas-Roberts
    Lester Hudson
    Paul George
    Shannon Brown
    Xavier Henry
    Victor Claver
    Zoran Dragic
    Sean Kilpatrick
    Glen Rice
    Darius Miller
    Andrei Kirilenko
    Patrick Christopher
    Bruno Caboclo
    Andre Dawkins
    Jamaal Franklin
    Tyrus Thomas
    Eric Moreland
    KJ McDaniels
    Timothy Hardaway Jr.
    JJ Hickson
    PJ Hairston
    Johnny OBryant
    DeMar DeRozan
    LeBron James
    Klay Thompson
    James Harden
    Kevin Durant
    Joe Johnson
    Paul George
    JR Smith
    Trevor Ariza
    Marcus Morris
    Kentavious Caldwell-Pope
    Gordon Hayward
    Luol Deng
    Khris Middleton
    Andrew Wiggins
    Wesley Matthews
    Harrison Barnes
    Kawhi Leonard
    Kyle Korver
    Al Farouq Aminu
    Tobias Harris
    Dion Waiters
    Patrick Patterson
    Nicolas Batum
    Courtney Lee
    Avery Bradley
    Marvin Williams
    Evan Fournier
    Justise Winslow
    Rodney Hood
    PJ Tucker
    Carmelo Anthony
    Jae Crowder
    Andre Iguodala
    Evan Turner
    Jimmy Butler
    Gary Harris
    Jeff Green
    Jabari Parker
    Kent Bazemore
    Allen Crabbe
    Rudy Gay
    Arron Afflalo
    Danny Green
    Matt Barnes
    Jamal Crawford
    Otto Porter Jr.
    Hollis Thompson
    Bojan Bogdanovic
    Devin Booker
    Terrence Ross
    Jerami Grant
    Andre Roberson
    Dante Cunningham
    Thabo Sefolosha
    Garrett Temple
    Taj Gibson
    Robert Covington
    Omri Casspi
    Aaron Gordon
    Kobe Bryant
    Doug McDermott
    Danilo Gallinari
    Randy Foye
    Nik Stauskas
    Chandler Parsons
    Luis Scola
    Stanley Johnson
    Corey Brewer
    Wesley Johnson
    Maurice Harkless
    Bradley Beal
    Richard Jefferson
    Iman Shumpert
    Shabazz Muhammad
    Marco Belinelli
    Gerald Green
    Gerald Henderson
    Alonzo Gee
    Wayne Ellington
    CJ Miles
    Jerryd Bayless
    Lance Stephenson
    Tayshaun Prince
    Ben McLemore
    Derrick Williams
    Mario Hezonja
    Devin Harris
    DeMarre Carroll
    Kyle Anderson
    Anthony Tolliver
    Luc Mbah a Moute
    Brandon Bass
    Leandro Barbosa
    Manu Ginobili
    Lance Thomas
    Mike Scott
    Tony Snell
    Paul Pierce
    Joe Ingles
    Jeremy Lamb
    Brandon Rush
    JaKarr Sampson
    Kevin Martin
    Marcus Thornton
    Vince Carter
    OJ Mayo
    TJ Warren
    Solomon Hill
    Nick Young
    Kyle Singler
    Anthony Morrow
    Rashad Vaughn
    Tim Hardaway Jr.
    Norman Powell
    Chase Budinger
    Sasha Vujacic
    Chris Johnson
    Luke Babbitt
    Gary Neal
    Tyreke Evans
    Justin Anderson
    James Anderson
    Mike Dunleavy
    Charlie Villanueva
    Kelly Oubre Jr.
    Sean Kilpatrick
    Rondae Hollis-Jefferson
    Anthony Brown
    Metta World Peace
    Glenn Robinson III
    James Jones
    Sonny Weems
    Troy Daniels
    John Jenkins
    Reggie Bullock
    Rasual Butler
    Lamar Patterson
    Sergey Karasev
    Darrun Hilliard
    Mike Miller
    James Ennis
    Axel Toupane
    Jordan Hamilton
    Kostas Papanikolaou
    Bryce Dejean-Jones
    Damjan Rudez
    Jeremy Evans
    Devyn Marble
    James Young
    Michael Kidd-Gilchrist
    Alan Anderson
    Caron Butler
    CJ Wilcox
    Elijah Millsap
    Damien Inglis
    Chris Copeland
    Cleanthony Early
    Pat Connaughton
    Jarell Eddie
    Aaron Harrison
    Anthony Bennett
    Alan Williams
    Jordan Mickey
    Josh Huestis
    Luis Montero
    Eric Moreland
    Jodie Meeks
    Steve Novak
    Bruno Caboclo
    Branden Dawson
    Duje Dukan
    Jimmer Fredette
    Jordan Adams
    Joe Harris
    Thanasis Antetokounmpo
    Sam Dekker
    Rakeem Christmas
    JJ Hickson
    PJ Hairston
    KJ McDaniels
    Johnny OBryant
    RJ Hunter
    J.J. OBrien
    LeBron James
    Klay Thompson
    Bradley Beal
    Trevor Ariza
    Giannis Antetokounmpo
    Andrew Wiggins
    Jimmy Butler
    Otto Porter Jr.
    DeMar DeRozan
    Marcus Smart
    Jae Crowder
    Gordon Hayward
    Kawhi Leonard
    Paul George
    Eric Gordon
    Nicolas Batum
    Kevin Durant
    Marcus Morris
    Andre Roberson
    Carmelo Anthony
    Kentavious Caldwell-Pope
    Tony Snell
    Wesley Matthews
    PJ Tucker
    Courtney Lee
    Andre Iguodala
    JJ Redick
    Victor Oladipo
    Thaddeus Young
    Solomon Hill
    Michael Kidd-Gilchrist
    Bojan Bogdanovic
    Allen Crabbe
    Maurice Harkless
    Joe Ingles
    Aaron Gordon
    Brandon Ingram
    Danny Green
    Evan Fournier
    JaMychal Green
    Wilson Chandler
    Joe Johnson
    Austin Rivers
    Danilo Gallinari
    Dario Saric
    Robert Covington
    Kent Bazemore
    Kyle Korver
    Monta Ellis
    TJ Warren
    DeMarre Carroll
    Luc Mbah a Moute
    Vince Carter
    Rodney McGruder
    Terrence Ross
    Jon Leuer
    Tony Allen
    Buddy Hield
    Rodney Hood
    CJ Miles
    Nikola Mirotic
    Matt Barnes
    E'Twaun Moore
    Richard Jefferson
    Kelly Oubre Jr.
    Evan Turner
    Gary Harris
    Marco Belinelli
    Jamal Murray
    Rondae Hollis-Jefferson
    Sean Kilpatrick
    Zach LaVine
    Jonathan Simmons
    JR Smith
    Gerald Henderson
    James Ennis
    Dante Cunningham
    Dorian Finney-Smith
    Jerami Grant
    Thabo Sefolosha
    Norman Powell
    Arron Afflalo
    Doug McDermott
    Jaylen Brown
    Nick Young
    Jeff Green
    Shabazz Muhammad
    Wayne Ellington
    Luol Deng
    Glenn Robinson III
    Sam Dekker
    Dirk Nowitzki
    Dion Waiters
    Stanley Johnson
    Jared Dudley
    Jonas Jerebko
    Ian Clark
    Corey Brewer
    Troy Daniels
    Caris LeVert
    Justin Anderson
    Kyle Anderson
    Nemanja Bjelica
    Timothé Luwawu-Cabarrot
    Ben McLemore
    Taurean Prince
    Mirza Teletovic
    Jeremy Lamb
    Brandon Knight
    Alex Abrines
    Khris Middleton
    Luke Babbitt
    Brandon Rush
    Mindaugas Kuzminskas
    Rudy Gay
    Michael Beasley
    Paul Zipser
    Lance Thomas
    Mario Hezonja
    Davis Bertans
    Mike Dunleavy
    Ron Baker
    Juan Hernangomez
    Wesley Johnson
    Tyreke Evans
    Hollis Thompson
    Anthony Morrow
    Jodie Meeks
    Gerald Green
    Isaiah Canaan
    Chandler Parsons
    Omri Casspi
    Justise Winslow
    Skal Labissière
    Troy Williams
    Marcus Thornton
    Derrick Jones Jr.
    Nicolas Brussino
    Okaro White
    Reggie Bullock
    Jordan Crawford
    James Jones
    Sasha Vujacic
    David Nwaba
    Kyle Singler
    Josh McRoberts
    Paul Pierce
    DeAndre Bembry
    Wayne Selden
    Pat Connaughton
    Maurice Ndour
    Damjan Rudez
    Alan Anderson
    Jake Layman
    James Young
    Archie Goodwin
    Cheick Diallo
    Malachi Richardson
    Treveon Graham
    Metta World Peace
    Anthony Brown
    Alex Poythress
    Mike Miller
    Henry Ellenson
    Jarrod Uthoff
    Ryan Kelly
    Georges Niang
    Alonzo Gee
    Reggie Williams
    Jarell Eddie
    Axel Toupane
    Dahntay Jones
    Patricio Garino
    Bruno Caboclo
    Josh Huestis
    Elijah Millsap
    Steve Novak
    Aaron Harrison
    John Jenkins
    AJ Hammons
    KJ McDaniels
    RJ Hunter
    LeBron James
    Klay Thompson
    Jrue Holiday
    Khris Middleton
    Andrew Wiggins
    Paul George
    Kevin Durant
    Jayson Tatum
    CJ McCollum
    DeMar DeRozan
    Joe Ingles
    JR Smith
    Trevor Ariza
    Josh Richardson
    Victor Oladipo
    Robert Covington
    Jaylen Brown
    Eric Gordon
    Bojan Bogdanovic
    Tobias Harris
    Harrison Barnes
    Otto Porter
    Taurean Waller-Prince
    JJ Redick
    Kentavious Caldwell-Pope
    Jae Crowder
    Kyle Kuzma
    Kelly Oubre
    Dillon Brooks
    Wilson Chandler
    Jimmy Butler
    Courtney Lee
    Gary Harris
    Yogi Ferrell
    Justin Holiday
    Marco Belinelli
    Al-Farouq Aminu
    Allen Crabbe
    Tony Snell
    DeMarre Carroll
    Bogdan Bogdanovic
    T.J. Warren
    Wesley Matthews
    Evan Turner
    Darius Miller
    Denzel Valentine
    Kyle Korver
    Marcus Smart
    Austin Rivers
    Kyle Anderson
    Jonathon Simmons
    Buddy Hield
    Andre Iguodala
    Marcus Morris
    Marvin Williams
    Lance Stephenson
    Nicolas Batum
    Brandon Ingram
    Jeremy Lamb
    Josh Jackson
    Stanley Johnson
    Danny Green
    Tim Hardaway
    Rodney Hood
    Caris LeVert
    Michael Kidd-Gilchrist
    Evan Fournier
    Ryan Anderson
    Justise Winslow
    Kent Bazemore
    Jerami Grant
    Jamal Crawford
    Doug McDermott
    Reggie Bullock
    OG Anunoby
    Luc Mbah a Moute
    Mario Hezonja
    Michael Beasley
    David Nwaba
    Troy Daniels
    Garrett Temple
    James Ennis
    Nick Young
    Bryn Forbes
    CJ Miles
    Dante Cunningham
    Pat Connaughton
    JaMychal Green
    Justin Jackson
    Wesley Johnson
    Luke Kennard
    Josh Hart
    Nemanja Bjelica
    Royce O'Neale
    Manu Ginobili
    Rudy Gay
    Semi Ojeleye
    Corey Brewer
    Lance Thomas
    Maurice Harkless
    Joe Johnson
    Alex Abrines
    Maxi Kleber
    Gerald Green
    Davis Bertans
    Sindarius Thornwell
    Jodie Meeks
    Norman Powell
    Ben McLemore
    Treveon Graham
    Andre Roberson
    Vince Carter
    Wesley Iwundu
    Tyler Dorsey
    Jerryd Bayless
    Dion Waiters
    Jabari Parker
    Sam Dekker
    Langston Galloway
    Malik Monk
    Paul Zipser
    Timothe Luwawu-Cabarrot
    Thabo Sefolosha
    Sterling Brown
    Omri Casspi
    DeAndre Liggins
    Cedi Osman
    Luke Babbitt
    CJ Williams
    Wayne Selden
    Chandler Parsons
    Jared Dudley
    Arron Afflalo
    Danilo Gallinari
    Zach LaVine
    Sean Kilpatrick
    Torrey Craig
    Terrence Ross
    Malik Beasley
    Brandon Paul
    Abdel Nader
    Justin Anderson
    Jonathan Isaac
    Kyle Collinsworth
    Damyean Dotson
    T.J. Leaf
    DeAndre' Bembry
    Dorian Finney-Smith
    Shabazz Muhammad
    Danuel House
    Ron Baker
    Myke Henry
    Glenn Robinson
    JaKarr Sampson
    Patrick Beverley
    Luke Kornet
    Malachi Richardson
    Antonio Blakeney
    Troy Williams
    Solomon Hill
    Jamel Artis
    Juan Hernangomez
    Jamil Wilson
    Tony Allen
    Derrick Jones
    Davon Reed
    Travis Wear
    Marcus Georges-Hunt
    Rashad Vaughn
    Kawhi Leonard
    Quincy Pondexter
    Jalen Jones
    John Holland
    Jake Layman
    Richard Jefferson
    Damien Wilkins
    Jon Leuer
    Malcolm Miller
    Antonius Cleveland
    James Webb
    Jabari Bird
    Bruno Caboclo
    Jaylen Morris
    Brice Johnson
    Darrun Hilliard
    Chris McCullough
    Okaro White
    James Young
    Kyle Singler
    Alfonzo McKinnie
    Vander Blue
    Charles Cooke
    Georges Niang
    Daniel Hamilton
    Jameel Warney
    Chinanu Onuaku
    James Michael McAdoo
    Matt Williams
    Nicolas Brussino
    Derrick Williams
    Luis Montero
    Vince Hunter
    Gordon Hayward
    Mindaugas Kuzminskas
    Naz Mitrou-Long
    Chris Boucher

 
Now that we have that data, let's save it so we don't need to scrape it again. 

**In [98]:**

{% highlight python %}
#wing_df.to_csv('wing_data.csv', index=False)
#guard_df.to_csv('guard_data.csv', index=False)
wing_df = pd.read_csv('wing_data.csv')
guard_df = pd.read_csv('guard_data.csv')
{% endhighlight %}
 
# Analysis

First, let's filter so we only have young guards and wings. We'll perform this
operation by finding all names of guards/wings with their `YRS_IN_LEAGUE` is 0. 

**In [99]:**

{% highlight python %}
young_wing_names = wing_df[wing_df['YRS_IN_LEAGUE']==0]['NAME']
young_guard_names = guard_df[guard_df['YRS_IN_LEAGUE']==0]['NAME']
{% endhighlight %}
 
## Plotting Statistical Improvement for all Players

Here, we plot the grade of **each** individual player for each skill. The data
will appear noisy, but it will give us a general idea of what we're looking at. 

**In [100]:**

{% highlight python %}
import matplotlib.pyplot as plt
import seaborn as sns
sns.set()
%matplotlib inline
{% endhighlight %}

**In [101]:**

{% highlight python %}
def plot_stat(stat, df, names, pos):
    l = stat.split('_')
    stat_name = ''
    for i in l:
        stat_name += i[0]+i[1:].lower()+' '
    stat_name+='Grade'
    plot_df = pd.DataFrame(columns=['Player', 'Year', stat_name])
    for player in names:
        p_df = df[df['NAME']==player]
        x = p_df['YEAR'].unique()
        y = p_df[stat]
        df2 = pd.DataFrame(columns=['Player', 'Year', stat_name])
        df2['Year'] = x
        df2[stat_name] = y.values
        df2['Player'] = player
        plot_df = plot_df.append(df2)
    fig, ax = plt.subplots(figsize=(50, 50))
    sns.pointplot(data=plot_df, x='Year', y=stat_name, hue='Player', ax=ax)
    ax.set_title(f'{stat_name} Improvement for all Young {pos} from 2014-2018', fontsize=50)
    ax.set_xlabel('Year', fontsize=40)
    ax.set_ylabel(f'{stat_name}', fontsize=40)
    fig.canvas.draw()
    ax.set_xticklabels(ax.get_xticklabels(), fontsize=30)
    ax.set_yticklabels(ax.get_yticklabels(), fontsize=30)
    ax.legend_.remove()
    ax.figure.savefig(f'{stat}_{pos}.png', bbox_inches='tight', pad_inches=1)
{% endhighlight %}

**In [102]:**

{% highlight python %}
wing_skills = ['PERIMETER_SHOT', 'OFF_BALL_MOVEMENT', 
                'ONE_ON_ONE', 'FINISHING', 'PERIMETER_DEFENSE', 'SELF_CREATION']
for s in wing_skills:
    plot_stat(s, wing_df, young_wing_names, 'Wings')
{% endhighlight %}

 
![png](https://i.imgur.com/QPjBIIY.png) 


 
![png](https://i.imgur.com/HVGB9cr.png) 


 
![png](https://i.imgur.com/jcn4SGz.png) 


 
![png](https://i.imgur.com/40TTsUq.png) 


 
![png](https://i.imgur.com/iZzul0u.png) 


 
![png](https://i.imgur.com/0rTiHqR.png) 


**In [103]:**

{% highlight python %}
guard_skills = ['PERIMETER_SHOT',
                   'ONE_ON_ONE', 'FINISHING', 'PLAYMAKING', 'SELF_CREATION']
for s in guard_skills:
    plot_stat(s, guard_df, young_guard_names, 'Guards')
{% endhighlight %}

 
![png](https://i.imgur.com/RPnSV4h.png) 


 
![png](https://i.imgur.com/x8g2zN4.png) 


 
![png](https://i.imgur.com/jKY2DWi.png) 


 
![png](https://i.imgur.com/WfNe50W.png) 


 
![png](https://i.imgur.com/tlyFo4b.png) 

 
Like I said, **very** noisy.

Let's clarify this data by finding the Year to Year difference in each skill for
each player. 
 
## Plotting Average Y2Y Improvement of a Certain Skill

Here, we take an individual `stat` and find the average year to year improvement
of an individual player.
Then, we average the year to year improvement for each `stat` and plot the
results. 

**In [104]:**

{% highlight python %}
def get_y2y_improvement(stat, df, names):
    d = {}
    for player in names:
        p_df = df[df['NAME']==player]
        p_df[f'{stat}_Y2Y_DIFF'] = p_df[stat].pct_change()
        p_df.fillna(0, inplace=True)
        x = p_df['YRS_IN_LEAGUE']
        y = p_df[f'{stat}_Y2Y_DIFF']
        for i in range(0, len(x.values)):
            yr = x.values[i]
            if yr not in d:
                d[yr] = []
            if y.values[i]==float('inf'):
                d[yr].append(0)
            else:
                d[yr].append(y.values[i]*100)
    for i in d:
        d[i] = sum(d[i])/len(d[i])
    return d
{% endhighlight %}

**In [105]:**

{% highlight python %}
wing_skills = ['PERIMETER_SHOT', 'OFF_BALL_MOVEMENT', 
                'ONE_ON_ONE', 'FINISHING', 'PERIMETER_DEFENSE', 'SELF_CREATION']
plot_df = pd.DataFrame(columns=['Experience', 'Y2Y Improvement', 'Skill'])
for s in wing_skills:
    d = get_y2y_improvement(s, wing_df, young_wing_names)
    df2 = pd.DataFrame(columns=['Experience', 'Y2Y Improvement', 'Skill'])
    df2['Experience'] = list(d.keys())
    df2['Y2Y Improvement'] = list(d.values())
    df2['Skill'] = s
    plot_df = plot_df.append(df2)
fig, ax = plt.subplots(figsize=(30, 30))
sns.pointplot(data=plot_df, x='Experience', y='Y2Y Improvement', hue='Skill')
ax.set_title(f'Average Year-to-year % Change in Wing Skills', fontsize=30)
ax.set_xlabel('Yrs Experience', fontsize=20)
ax.set_ylabel(f'Average Y2Y % Change', fontsize=20)
fig.canvas.draw()
ax.set_xticklabels(ax.get_xticklabels(), fontsize=15)
ax.set_yticklabels(ax.get_yticklabels(), fontsize=15)
plt.setp(ax.get_legend().get_texts(), fontsize=30) # for legend text
plt.setp(ax.get_legend().get_title(), fontsize=40) # for legend title
ax.figure.savefig(f'wing_y2y.png', bbox_inches='tight', pad_inches=1)
{% endhighlight %}

    /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/ipykernel_launcher.py:5: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      """


 
![png](https://i.imgur.com/SvdnjAC.png) 

 
## Wing Takeaways
* Note the scale on this grid! Evidently, wings improve **MASSIVELY**
continuously, but peter out by approximately year 3
* Interesting how much improvement their is in each skill. Off ball movement
appears to be the most significant change upon arriving in the league and
continues to be throughout the first couple years

 

**In [106]:**

{% highlight python %}
guard_skills = ['PERIMETER_SHOT',
                   'ONE_ON_ONE', 'FINISHING', 'PLAYMAKING', 'SELF_CREATION']
plot_df = pd.DataFrame(columns=['Experience', 'Y2Y Improvement', 'Skill'])
for s in guard_skills:
    d = get_y2y_improvement(s, guard_df, young_guard_names)
    df2 = pd.DataFrame(columns=['Experience', 'Y2Y Improvement', 'Skill'])
    df2['Experience'] = list(d.keys())
    df2['Y2Y Improvement'] = list(d.values())
    df2['Skill'] = s
    plot_df = plot_df.append(df2)
fig, ax = plt.subplots(figsize=(30, 30))
sns.pointplot(data=plot_df, x='Experience', y='Y2Y Improvement', hue='Skill')
ax.set_title(f'Average Year-to-year % Change in Guard Skills', fontsize=30)
ax.set_xlabel('Yrs Experience', fontsize=20)
ax.set_ylabel(f'Average Y2Y % Change', fontsize=20)
fig.canvas.draw()
ax.set_xticklabels(ax.get_xticklabels(), fontsize=15)
ax.set_yticklabels(ax.get_yticklabels(), fontsize=15)
plt.setp(ax.get_legend().get_texts(), fontsize=30) # for legend text
plt.setp(ax.get_legend().get_title(), fontsize=40) # for legend title
ax.figure.savefig(f'guard_y2y.png', bbox_inches='tight', pad_inches=1)
{% endhighlight %}

    /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/ipykernel_launcher.py:5: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      """


 
![png](https://i.imgur.com/mUuScmS.png) 

 
## Guard Takeaways

* Note the scale on this graph - **NOT** as significant!
* Once again, guards also peter out by year 3
* Guards seem to improve most at finishing and one-on-one play 
 
# Conclusions

According my plots above, the takeaway is fairly clear. Simply the scale on the
plots seems to indicate that the majority of the improvement occurs between
years 0-3. In this timespan, wings improve **greatly** at their most relevant
skills, whereas guards do not.

As a result, I think it is reasonable to conclude that **GUARDS** are the more
difficult to develop than wings. Overall, wings seem to improve more easily than
guards as indicated by the scales on the above plots. 

**In [None]:**

{% highlight python %}

{% endhighlight %}
