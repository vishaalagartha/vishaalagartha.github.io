---
title: "Basketball Reference Scraper"
date: 2024-03-18
permalink: /projects/2024/03/18/basketball-reference-scraper
tags:
    - Projects
--- 

NBA stats and analytics are growing as a field and becoming more and more popular among young programmers seeking entry into fields of data analysis, machine learning, and general statistics.

However, the barrier to entry often doesn't have to do with libraries involving the actual analytics like scikit learn, tensorflow, or pytorch. People don't have easy access to data.

Libraries like [nba_api](https://github.com/swar/nba_api) exist, but the endpoints (in my opinion) are rather confusing and complicated. How can I just a given draft class? Which endpoint is it in `draftboard`? `draftcombinedrillresults`? `draftcombinenonstationaryshooting`?
`draftcombineplayeranthro`? `draftcombinespotshooting`? `draftcombinestats`? `drafthistory`?

Basketball Reference provides these stats in a simpler view **with a UI** users can see. Want to see what information you'll get? Just go to the page [https://www.basketball-reference.com/draft/NBA_2022.html](https://www.basketball-reference.com/draft/NBA_2022.html) and see for yourself.

Rather than having to write your own scraper, I created a Python package that allows users to abstract away all the complicated logic of handling rate limits, dynamic content, etc. 

Note that this package existed prior to [Sports Reference turning off their widgets](https://www.sports-reference.com/blog/2022/10/sports-reference-will-turn-off-our-widgets-service-january-1-2023/). The process was much simpler and we would not need to manually scrape the website. Additionally, they recently [added rate limiters to prevent excess scraping](https://www.sports-reference.com/bot-traffic.html).

[Basketball Reference Scraper 2.0](https://github.com/vishaalagartha/basketball_reference_scraper/blob/master/API.md) builds on the existing foundation and leverages Selenium to dynamically scrape content from the site while ensuring users never hit the rate limit threshold.

Some functions the package offers are:
* `get_roster(team, season)`
* `get_stats(name, stat_type='PER_GAME', playoffs=False, career=False)`
* `get_player_headshot(name)`
* `get_box_scores(date, team1, team2, period='GAME', stat_type='BASIC')`
* `get_pbp(date, team1, team2)`

And many more!

The main modules the package offers are:
* `teams`
* `players`
* `seasons`
* `box_scores`
* `pbp` (Play-by-play)
* `shot_charts`
* `injury_report`
* `drafts`

Please refer to the full [API documentation](https://github.com/vishaalagartha/basketball_reference_scraper/blob/master/API.md) for more details.