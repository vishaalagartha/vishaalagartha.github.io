---
title: "Computing Shapley Values in the NBA"
date: 2019-09-01
permalink: /projects/2019/09/01/nba-shapley
--- 

NOTE: all the code for this project is publicly available at [this repo](https://github.com/vishaalagartha/nba-shapley).

For the more visually inclined, check out this [circle-packing visualization](https://bl.ocks.org/vishaalagartha/raw/6dd6fa8641ed6eda10913c90b6e4c91a/) to see a players contribution!
 * Choose the preferred statistic at the top right (currently we have Net Rating, Offensive Rating, and Defensive Rating)
 * Choose the preferred lineup directly below. Note that the lineups are sorted by rank!

## Introduction
Basketball statistics are flawed in that they are not placed in a context. Even advanced statistics like PER, Real Plus Minus, eFG%, etc. are computed via contrived formulas.

Basketball is, inherently, a team sport, and ignoring the fact that player X's presence affects player Y's performanceis a mistake. 

So how can we capture the contribution each player has on a team?

## Cooperative Game Theory

In game theory, certain games are classified as [cooperative games](ttps://en.wikipedia.org/wiki/Cooperative_game_theory) or games where players work together to achieve an end. Think of the boardgame Spaceteam or Pandemic. However, cooperative game theory also applies in the real world as well - like in the situation of companies or... sports teams.

## The Shapley Value

In 1953, Lloyd Shapley introduced the concept of a [Shapley value](https://en.wikipedia.org/wiki/Shapley_value) for cooperative games. This value aims to attribute values to invidividuals on a team and analyze the marginal contribution each player adds to a team.

The Shapley value has been used in many real world settings such as politics, economics, business, and marketing. It has even been explored in the world of [soccer](https://northyardanalytics.com/shapley-values-english-premier-league-2012-13.php). However, it has never been applied in the NBA to see how much an individual contributes to a team.

## Methodology
Of course, the Shapley value can be used on any given metric (i.e. wins, PPG, Offensive Rating, Defensive Rating, etc.). But, this methodology decided to go with a go-to method for analyzing 5 man lineups: Net Rating per 100 possessions.

I decided to go off of this metric since it seemed to represent which 5 man lineups are best. You can see for yourself on by creating a custom filter on the nba stats website [here](https://stats.nba.com/lineups/advanced/?sort=NET_RATING&dir=1&CF=MIN*GE*100). Evidently, the Hampton's 5 have been commonly called the greatest 5 man lineup of all time and this statistic clearly reflects that. We can also see that several of the other most dominant lineups such as Toronto's starting 5 and Philadelphia's starting 5 show up near the top of the list.

Oh and also it helps the [Zach Lowe thinks of it as a valuable metric](https://twitter.com/zachlowe_nba/status/969255248723431426?lang=en).

### Data Aggregation

I decided to test on all 5 man lineups that have logged at least 100 minutes in the 2018-19 season. This was acquired simply using Python's requests library and fetching data using the [leaguedashlineups endpoint](http://stats.nba.com/stats/leaguedashlineups/) from the NBA stats API and adding the required parameters. 

You'll notice that in order to compute Shapley values, we need to accumulate the Net Ratings of **each subset of players in the lineup**. So if you want to compute the Shapley Values of the Hampton's 5, you need the Net Ratings of the following subsets:

```
['Stephen Curry']
['Andre Iguodala']
['Klay Thompson']
['Kevin Durant']
['Draymond Green']
['Stephen Curry', 'Andre Iguodala']
['Stephen Curry', 'Klay Thompson']
['Stephen Curry', 'Kevin Durant']
['Stephen Curry', 'Draymond Green']
['Andre Iguodala', 'Klay Thompson']
['Andre Iguodala', 'Kevin Durant']
['Andre Iguodala', 'Draymond Green']
['Klay Thompson', 'Kevin Durant']
['Klay Thompson', 'Draymond Green']
['Kevin Durant', 'Draymond Green']
['Stephen Curry', 'Andre Iguodala', 'Klay Thompson']
['Stephen Curry', 'Andre Iguodala', 'Kevin Durant']
['Stephen Curry', 'Andre Iguodala', 'Draymond Green']
['Stephen Curry', 'Klay Thompson', 'Kevin Durant']
['Stephen Curry', 'Klay Thompson', 'Draymond Green']
['Stephen Curry', 'Kevin Durant', 'Draymond Green']
['Andre Iguodala', 'Klay Thompson', 'Kevin Durant']
['Andre Iguodala', 'Klay Thompson', 'Draymond Green']
['Andre Iguodala', 'Kevin Durant', 'Draymond Green']
['Klay Thompson', 'Kevin Durant', 'Draymond Green']
['Stephen Curry', 'Andre Iguodala', 'Klay Thompson', 'Kevin Durant']
['Stephen Curry', 'Andre Iguodala', 'Klay Thompson', 'Draymond Green']
['Stephen Curry', 'Andre Iguodala', 'Kevin Durant', 'Draymond Green']
['Stephen Curry', 'Klay Thompson', 'Kevin Durant', 'Draymond Green']
['Andre Iguodala', 'Klay Thompson', 'Kevin Durant', 'Draymond Green']
['Stephen Curry', 'Andre Iguodala', 'Klay Thompson', 'Kevin Durant', 'Draymond Green']
```

This is the powerset of the lineup and results in 2^5-1 total subsets.

To achieve this, I used, once again the NBA stats [leaguedashplayerstats endpoint](http://stats.nba.com/stats/leaguedashplayerstats/?) with the following parameters:
```
params = {
'measureType': 'Advanced',
'perMode': 'Per100Possessions',
'paceAdjust': 'N',
'plusMinus': 'Y',
'rank': 'N',
'leagueId': '00',
'season': None,
'seasonType': 'Regular Season',
'poRound': '0',
'outcome': '',
'location': '',
'month': '0',
'seasonSegment': '',
'dateFrom': '',
'dateTo': '',
'opponentTeamId': '0',
'vsConference': '',
'vsDivision': '',
'gameScope': '',
'playerExperience': '',
'playerPosition': '',
'starterBench': '',
'draftYear': '',
'gameSegment': '',
'period': '0',
'lastNGames': '0',
'draftYear': '',
'draftPick': '',
'college': '',
'country': '',
'height': '',
'weight': ''
}
```

Here, I set `season: 2018-19`. This gave me the Net Ratings when an **individual** player was on the court.

To acquire data for group sizes of 2, 3, 4, and 5, I used the [leaguedashlineups endpoint](http://stats.nba.com/stats/leaguedashlineups/?) with the following parameters:
```
params = {
'measureType': 'Advanced',
'perMode': 'Per100Possessions',
'paceAdjust': 'N',
'plusMinus': 'Y',
'rank': 'N',
'leagueId': '00',
'season': None,
'seasonType': 'Regular Season',
'poRound': '0',
'outcome': '',
'location': '',
'month': '0',
'seasonSegment': '',
'dateFrom': '',
'dateTo': '',
'opponentTeamId': '0',
'vsConference': '',
'vsDivision': '',
'teamId': None,
'conference': '',
'division': '',
'gameSegment': '',
'period': '0',
'shotClockRange': '',
'lastNGames': '0',
'groupQuantity': None,
}
```

With `season: 2018-19` and appropriate `teamId` and `groupQuantity` values.

## Results
Using the acquired data, I was able to compute Shapley values for the best lineups during the 2018-19 season. Here are some of the results for the top 5 lineups according to Net Rating Per 100 Possessions:
```
[('Stephen Curry', 8.06666666666667), ('Andre Iguodala', 7.733333333333333), ('Draymond Green', 7.066666666666675), ('Kevin Durant', 4.974999999999995), ('Klay Thompson', 0.15833333333333416)]

[('Paul George', 8.528333333333325), ('Dennis Schroder', 4.945000000000004), ('Steven Adams', 4.5366666666666635), ('Terrance Ferguson', 4.436666666666661), ('Jerami Grant', 2.753333333333332)]

[('Malik Beasley', 9.559999999999999), ('Paul Millsap', 5.735000000000001), ('Torrey Craig', 2.843333333333335), ('Nikola Jokic', 2.760000000000001), ('Jamal Murray', 1.901666666666668)]

[('Davis Bertans', 12.12833333333333), ('Derrick White', 8.294999999999982), ('DeMar DeRozan', 1.2949999999999995), ('LaMarcus Aldridge', 0.09499999999999963), ('Bryn Forbes', 0.086666666666666)]

[('Davis Bertans', 8.340000000000009), ('Patty Mills', 4.498333333333332), ('Rudy Gay', 4.014999999999996), ('Jakob Poeltl', 3.923333333333329), ('Marco Belinelli', 1.0233333333333328)]
```


Here is a waterfall plot of what the Hampton's 5 looks like:
![alt text](https://i.imgur.com/GC4oYYn.png)

We can clearly see that Stephen Curry contributes the most, as expected. Interestingly enough, Andre Iguodala and Drymond Green contribute the next most, despite not being the box score fillers. Rather, we see that the typical fillers of Klay Thompson and Kevin Durant contribute much less.

This goes in line with what many NBA stats gurus say and observe. They see that when KD and Klay are on the court, they add scoring. But is this scoring necessary for the Hampton's 5's success? Or are the defensive benefits, passing ability, and overall basketball IQ provided by Iguodala and Green more beneficial? 

## Conclusions

Evidently, the Shapley value goes beyond the surface level many people base their judgement of a player on. Of course, the metric is not perfect and needs more nuancing. Possible improvements/tests include:

- Modifying the filter for lineups
- Testing out different metrics (i.e. box plus minus, per game, etc.)
- Trying to calculate the Shapley value for an entire team (not just a 5 man lineup)
- Using different methods of aggregation

Once again, all the code can be found at [this repo](https://github.com/vishaalagartha/nba-shapley).
For the more visually inclined, check out this [circle-packing visualization](https://bl.ocks.org/vishaalagartha/raw/6dd6fa8641ed6eda10913c90b6e4c91a/) to see a players contribution!
  - Choose the preferred statistic at the top right (currently we have Net Rating, Offensive Rating, and Defensive Rating)
  - Choose the preferred lineup directly below. Note that the lineups are sorted by rank!
