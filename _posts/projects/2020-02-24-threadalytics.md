---
title: "threadalytics - A Game Thread Analysis Engine"
date: 2020-02-24
permalink: /projects/2020/02/24/threadalytics
tags:
    - python
    - notebook
--- 

For proprietary reasons, I will not be posting an explanation on the creation of this site.

This is a long term project that I plan on expanding, but here is the basic premise:

I made a web application to analyze game threads. Here's a link: [https://threadalytics.herokuapp.com/](https://threadalytics.herokuapp.com/)

Threadalytics: A Game Thread Analysis Engine

Hey everyone,

I recently made a web application that scrapes Game Thread comments and performs some analysis on the comments. Some features of the application include:

* Basic summary statistics (number of comments, authors, etc)
* A word cloud that correlates word size with frequency
* A graph displaying comment frequency over time. Upon hovering on an individual data point, a user can see some relevant comments to get an idea of what caused spikes in activity.
* A graph displaying sentiment over time. Once again, upon hovering on an individual data point, a user can see comments that caused such a positive or negative sentiment.
* F\*CK statistics - who dropped the f-bomb the most? what were some of the most prolific bomb drops?
* Ref complaints - did people complain about the refs? if so, who did the most?
* The Thread's MVP and runners up
* Happiest Author (with comments that made him/her happy)
* Saddest Author (with comments that made him/her sad)

Here is a link to the application:

[https://threadalytics.herokuapp.com](https://threadalytics.herokuapp.com/games/f1if9o)

Here is a link to the warriors homepage:

[https://threadalytics.herokuapp.com/teams/GSW/](https://threadalytics.herokuapp.com/teams/GSW/)

And here is a link to the game against the heat:

[https://threadalytics.herokuapp.com/teams/GSW/games/f223ap](https://threadalytics.herokuapp.com/teams/GSW/games/f223ap)

Upon multiple requests, I expanded to also include leaderboards for every team's subreddit so that you can view the following statistics for the entire season:
* Average Compound Sentiment Scores for all commenters
* Average Positive Sentiment Scores for all commenters
* Average Negative Sentiment Scores for all commenters
* Number of comments
* Total Score (upvotes-downvotes)
* F*CK Count
* Referee References

You can view these statistics here on the [leaderboard page](https://threadalytics.herokuapp.com/leaderboard/GSW).

Finally, here is the [comparison page](https://threadalytics.herokuapp.com/compare) to compare statistics across all r/nba subreddits. Here, you can see:

* The total number of comments by subreddit
* Box plots of compound sentiments by subreddit
* Donut chart of most F\* Bombs dropped along with numbers
* Donut chart of most Ref References along with numbers

Note that it may not render that well on mobile! I'm working on fixing this.

