---
title: "Analyzing Voter Fatigue"
date: 2020-01-28
permalink: /drafts/voter-fatigue
tags:
    - python
    - notebook
--- 


## Introduction

Legacies in the NBA are defined by 2 items more than anything:
* Championships
* Awards

Whereas championships are earned purely objectively through competition, awards present a more *subjective* element. As a result, the number of awards a player receives is more prone to bias.

A landmark moment to illustrate this pattern was in 2011 when Derrick Rose won the MVP over Lebron James. 

Lebron James was coming off of an incredible year in Miami with just around 27 points, 7.5 rebounds, and 7 assists per game that season with a field goal percentage of 51%. In comparison to Rose’s 25 points, 4 rebounds, and 8 assists with a field goal percentage of 45%, Lebron had the better statistics for that season. But, Lebron had just come off of back-to-back MVP's, so who would want to vote for him again.

This tendency to favor the new candidate over the previous year's winner is known as **voter fatigue**. It is a pattern that occurs for individual awards such as the MVP, DPOY, MIP, SMOY, and COY. 

So, how much does voter fatigue matter for each of the individual awards? This project aims to investigate how much *better* a winner of each individual award must be the following year to repeat.

## Methodology

### Measuring A Candidate's Success
For each award we choose the most relevant statistic that determines the award.

For the MVP, we choose Win Shares (WS) since the MVP should be determined by the player who contributes most to the team's success.

For DPOY, we choose Defensive Win Shares (DFS) since the DPOY should be determined by the player who contributes the most to the team's success *exclusively on the defensive end*.

For MIP, we choose the difference between the player's PER the year they were selected and the player's PER the year prior.

For SMOY, we choose the player's PER that year.

For COY, we choose the difference between the number of wins the team won that year and the number of wins the team was predicted to win ([Pythagorean Wins](https://www.basketball-reference.com/about/glossary.html) on Basketball Reference).

To summarize:

|Award|Metric|
|--|--|
|MVP|WS|
|DPOY|DWS|
|MIP|CURR_PER-PREV_PER|
|SMOY|PER|
|COY|W-PW|

Of course, we will scale each metric appropriately to land between 0.0 and 1.0 so we can compare between all awards.

### Measuring A Candidate's Award Voting

For MVP, DPOY, MIP, and SMOY, we aggregate the the share of the points that the player received during voting from the years 1988 onward since award statistics were first released in 1988.

Since the points share is always scaled between 0.0 and 1.0, we will be able to compare between awards appropriately.

Unfortunately, award statistics were only released in 2016 for COY, so our dataset will be much smaller.

### Computing Voter Fatigue

For each award, we compute the difference between the points share the candidate received the following year. Next, we compute the difference between the relevant scaled statistic.

For example (note that WS is not scaled in this example for clarity):

Michael Jordan won the MVP in 1988 with a WS of 21.2 and a PTS_SHARE of 0.831.
In 1989, he lost to Magic Johnson, with a WS of 19.8 and a PTS_SHARE of 0.704.

So the two data points we compute are 21.2-19.8 = 0.4, and 0.831-0.704=0.127

We then plot the difference in the Points Share as a function of the statistic's difference and fit a regression line.

Finally, we find the y-intercept of this regression line.

This y-intercept measures how much better the candidate must perform **if they were to maintain their performance from their previous year AND receive the same points share**.

Using this value, we can quantify voter fatigue for each award.

Let's begin!
