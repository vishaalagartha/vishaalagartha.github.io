---
title: "The Basketball Reference Scraper"
date: 2020-01-15
permalink: /posts/2020/01/15/bbref-scraper
tags:
    - python
    - notebook
--- 
## Introduction

As an NBA analytics enthusiast, I need to be able to get my hands on one thing:
data. NBA data can be acquired from a few locations:
  - [NBA Stats](https://www.stats.nba.com)
  - [Basketball Reference](https://www.basketball-reference.com)
  - [ESPN](https://www.espn.com)
  - ...

The problem with most of these websites is their extremely confusing and
contrived endpoints. Consider the most popular site [NBA
Stats](https://www.stats.nba.com). Here are just a few of their endpoints:
  - `allstarballotpredictor`
  - `boxscoreadvancedv2`
  - `boxscorefourfactorsv2`
  - `boxscoremiscv2`
  - ...

Who needs all of this and what do each of them mean?! Moreover, take a look at
what they return for a simple endpoint `commonallplayers`:
```
['PERSON_ID', 'DISPLAY_LAST_COMMA_FIRST', 'DISPLAY_FIRST_LAST', 'ROSTERSTATUS',
'FROM_YEAR', 'TO_YEAR', 'PLAYERCODE', 'TEAM_ID', 'TEAM_CITY', 'TEAM_NAME',
'TEAM_ABBREVIATION', 'TEAM_CODE', 'GAMES_PLAYED_FLAG',
'OTHERLEAGUE_EXPERIENCE_CH']
```
I don't need all this information.

Additionally, I've found that a lot of sites including stats.nba.com prevent
multiple repeated requests from the same IP, making the simple data acquisition
process heinous.

## The Solution

Basketball Reference, on the other hand, does not pose such a problem.

Basketball Reference provides clear data output and does not. For all those
questioning whether this is in compliance with their ToS, they explicitly state:

'As an aside, copyright law is clear that facts cannot be copyrighted, so you
are free to reuse facts found on this site in accordance with copyright laws.'

Additionally, after stating that you shouldn't scrape the data, they tell you
that you should scrape the data:

'However, I would point out that learning how to accumulate data is often a more
valuable skill than actually analyzing the data, so we encourage you as a
student or professional to learn how.' 
 
## So how does one go about achieving this?

Typically, scraping is permormed on static websites using the Python `requests`
library and `BeautifulSoup`. Consider the following example of scraping my own
[homepage](https://www.vishaalagartha.github.io): 

**In [2]:**

{% highlight python %}
# Import relevant libraries
from requests import get
from bs4 import BeautifulSoup

r = get('https://vishaalagartha.github.io')
if r.status_code==200:
    soup = BeautifulSoup(r.content, 'html.parser')
    print(soup.prettify())
{% endhighlight %}

    <!DOCTYPE html>
    <html class="no-js" lang="en">
     <head>
      <meta charset="utf-8"/>
      <!-- begin SEO -->
      <title>
       Vishaal Agartha
      </title>
      <meta content="en-US" property="og:locale"/>
      <meta content="Vishaal Agartha" property="og:site_name"/>
      <meta content="Vishaal Agartha" property="og:title"/>
      <link href="https://vishaalagartha.github.io/" rel="canonical"/>
      <meta content="https://vishaalagartha.github.io/" property="og:url"/>
      <meta content="About me" property="og:description"/>
      <script type="application/ld+json">
       { "@context" : "http://schema.org", "@type" : "Person", "name" : "Vishaal Agartha", "url" : "https://vishaalagartha.github.io", "sameAs" : null }
      </script>
      <!-- end SEO -->
      <link href="https://vishaalagartha.github.io/feed.xml" rel="alternate" title="Vishaal Agartha Feed" type="application/atom+xml"/>
      <!-- http://t.co/dKP3o1e -->
      <meta content="True" name="HandheldFriendly"/>
      <meta content="320" name="MobileOptimized"/>
      <meta content="width=device-width, initial-scale=1.0" name="viewport"/>
      <script>
       document.documentElement.className = document.documentElement.className.replace(/\bno-js\b/g, '') + ' js ';
      </script>
      <!-- For all browsers -->
      <link href="https://vishaalagartha.github.io/assets/css/main.css" rel="stylesheet"/>
      <meta content="on" http-equiv="cleartype"/>
      <!-- start custom head snippets -->
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-57x57.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="57x57"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-60x60.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="60x60"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-72x72.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="72x72"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-76x76.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="76x76"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-114x114.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="114x114"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-120x120.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="120x120"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-144x144.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="144x144"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-152x152.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="152x152"/>
      <link href="https://vishaalagartha.github.io/images/apple-touch-icon-180x180.png?v=M44lzPylqQ" rel="apple-touch-icon" sizes="180x180"/>
      <link href="https://vishaalagartha.github.io/images/favicon-32x32.png?v=M44lzPylqQ" rel="icon" sizes="32x32" type="image/png"/>
      <link href="https://vishaalagartha.github.io/images/android-chrome-192x192.png?v=M44lzPylqQ" rel="icon" sizes="192x192" type="image/png"/>
      <link href="https://vishaalagartha.github.io/images/favicon-96x96.png?v=M44lzPylqQ" rel="icon" sizes="96x96" type="image/png"/>
      <link href="https://vishaalagartha.github.io/images/favicon-16x16.png?v=M44lzPylqQ" rel="icon" sizes="16x16" type="image/png"/>
      <link href="https://vishaalagartha.github.io/images/manifest.json?v=M44lzPylqQ" rel="manifest"/>
      <link color="#000000" href="https://vishaalagartha.github.io/images/safari-pinned-tab.svg?v=M44lzPylqQ" rel="mask-icon"/>
      <link href="/images/favicon.ico?v=M44lzPylqQ" rel="shortcut icon"/>
      <meta content="#000000" name="msapplication-TileColor"/>
      <meta content="https://vishaalagartha.github.io/images/mstile-144x144.png?v=M44lzPylqQ" name="msapplication-TileImage"/>
      <meta content="https://vishaalagartha.github.io/images/browserconfig.xml?v=M44lzPylqQ" name="msapplication-config"/>
      <meta content="#ffffff" name="theme-color"/>
      <link href="https://vishaalagartha.github.io/assets/css/academicons.css" rel="stylesheet">
       <script type="text/x-mathjax-config">
        MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } });
       </script>
       <script type="text/x-mathjax-config">
        MathJax.Hub.Config({ tex2jax: { inlineMath: [ ['$','$'], ["\\(","\\)"] ], processEscapes: true } });
       </script>
       <script async="" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/latest.js?config=TeX-MML-AM_CHTML">
       </script>
       <!-- end custom head snippets -->
      </link>
     </head>
     <body>
      <!--[if lt IE 9]><div class="notice--danger align-center" style="margin: 0;">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</div><![endif]-->
      <div class="masthead">
       <div class="masthead__inner-wrap">
        <div class="masthead__menu">
         <nav class="greedy-nav" id="site-nav">
          <button>
           <div class="navicon">
           </div>
          </button>
          <ul class="visible-links">
           <li class="masthead__menu-item masthead__menu-item--lg">
            <a href="https://vishaalagartha.github.io/">
             Vishaal Agartha
            </a>
           </li>
           <li class="masthead__menu-item">
            <a href="https://vishaalagartha.github.io/about/">
             About Me
            </a>
           </li>
           <li class="masthead__menu-item">
            <a href="https://vishaalagartha.github.io/projects/">
             Projects
            </a>
           </li>
           <li class="masthead__menu-item">
            <a href="https://vishaalagartha.github.io/notes/">
             Notes
            </a>
           </li>
           <li class="masthead__menu-item">
            <a href="https://vishaalagartha.github.io/rockclimbing/">
             Rockclimbing
            </a>
           </li>
           <li class="masthead__menu-item">
            <a href="https://vishaalagartha.github.io/resume/">
             Resume
            </a>
           </li>
          </ul>
          <ul class="hidden-links hidden">
          </ul>
         </nav>
        </div>
       </div>
      </div>
      <div id="main" role="main">
       <div class="sidebar sticky">
        <div itemscope="" itemtype="http://schema.org/Person">
         <div class="author__avatar">
          <img alt="Vishaal Agartha" class="author__avatar" src="https://vishaalagartha.github.io/images/profile.jpg"/>
         </div>
         <div class="author__content">
          <h3 class="author__name">
           Vishaal Agartha
          </h3>
          <p class="author__bio">
           Frontend Engineer for Trifecta | Blockchain Engineer | NBA Analytics Enthusiast | Obsessed Boulderer
          </p>
         </div>
         <div class="author__urls-wrapper">
          <button class="btn btn--inverse">
           Follow
          </button>
          <ul class="author__urls social-icons">
           <li>
            <i aria-hidden="true" class="fa fa-fw fa-map-marker">
            </i>
            Berkeley, CA
           </li>
           <li>
            <i aria-hidden="true" class="fa fa-fw fa-map-marker">
            </i>
            Trifecta
           </li>
           <li>
            <a href="mailto:vishaalagartha@gmail.com">
             <i aria-hidden="true" class="fas fa-fw fa-envelope">
             </i>
             Email
            </a>
           </li>
           <li>
            <a href="https://www.linkedin.com/in/vishaalagartha">
             <i aria-hidden="true" class="fab fa-fw fa-linkedin">
             </i>
             LinkedIn
            </a>
           </li>
           <li>
            <a href="https://instagram.com/vishaalagartha">
             <i aria-hidden="true" class="fab fa-fw fa-instagram">
             </i>
             Instagram
            </a>
           </li>
           <li>
            <a href="https://github.com/vishaalagartha">
             <i aria-hidden="true" class="fab fa-fw fa-github">
             </i>
             Github
            </a>
           </li>
          </ul>
         </div>
        </div>
       </div>
       <article class="page" itemscope="" itemtype="http://schema.org/CreativeWork">
        <meta content="" itemprop="headline"/>
        <meta content="About me" itemprop="description"/>
        <div class="page__inner-wrap">
         <header>
          <h1 class="page__title" itemprop="headline">
          </h1>
         </header>
         <section class="page__content" itemprop="text">
          <h1 id="hi-my-name-is-vishaal-agartha">
           Hi, my name is Vishaal Agartha.
          </h1>
          <h1 id="welcome-to-my-corner-of-the-internet">
           Welcome to my corner of the Internet.
          </h1>
          <p>
           I’m a 22 year old computer scientist who recently graduated from UCLA. I’m currently working as a Frontend and Data Visualization Engineer at
           <a href="www.trifecta.com">
            Trifecta Inc
           </a>
           . I recently applied for a Master’s program and aspire to become a data scientist and machine learning engineer. My other passions include rockclimbing and NBA basketball analytics.
          </p>
          <p>
           On the rockclimbing spectrum, I’m primarily a boulderer. Bouldering consists of short ‘problems’ compared to sport climbing or even big wall climbing. Routes are typically 20-30 ft off the ground and require anywhere from 10 seconds to a minute of intense movement.
          </p>
          <p>
           Although I enjoy data visualization and front end engineering, my ultimate career goal is NBA sports analytics. Hence, I spend a lot of my free time trying to find interesting patterns and insights into the wild, dynamic sport of basketball. My dream job is to work in the cross-section of computer science and sports. I believe the NBA is undergoing a technological revolution and teams can gain an edge by analyzing large amounts of data via artificial intelligence and machine learning algorithms.
          </p>
          <p>
           If you’re looking for a summary of me and my qualifications please refer to my
           <a href="https://vishaalagartha.github.io/resume">
            Resume
           </a>
           .
          </p>
          <p>
           If you want to check out my latest projects, please check out the
           <a href="https://vishaalagartha.github.io/projects">
            Projects
           </a>
           page.
          </p>
          <p>
           If you’re a rockclimbing enthusiast, check out my
           <a href="https://vishaalagartha.github.io/rockclimbing">
            Rockcliming
           </a>
           page.
          </p>
          <p>
           Finally, if you want to peek into my cluttered brain dump, look at my
           <a href="https://vishaalagartha.github.io/notes">
            Notes
           </a>
           .
          </p>
         </section>
         <footer class="page__meta">
         </footer>
        </div>
       </article>
      </div>
      <div class="page__footer">
       <footer>
        <!-- start custom footer snippets -->
        <a href="/sitemap/">
         Sitemap
        </a>
        <!-- end custom footer snippets -->
        <div class="page__footer-follow">
         <ul class="social-icons">
          <li>
           <strong>
            Follow:
           </strong>
          </li>
          <li>
           <a href="http://github.com/vishaalagartha">
            <i aria-hidden="true" class="fab fa-github">
            </i>
            GitHub
           </a>
          </li>
          <li>
           <a href="https://vishaalagartha.github.io/feed.xml">
            <i aria-hidden="true" class="fa fa-fw fa-rss-square">
            </i>
            Feed
           </a>
          </li>
         </ul>
        </div>
        <div class="page__footer-copyright">
         © 2020 Vishaal Agartha. Powered by
         <a href="http://jekyllrb.com" rel="nofollow">
          Jekyll
         </a>
         &amp;
         <a href="https://github.com/academicpages/academicpages.github.io">
          AcademicPages
         </a>
         , a fork of
         <a href="https://mademistakes.com/work/minimal-mistakes-jekyll-theme/" rel="nofollow">
          Minimal Mistakes
         </a>
         .
        </div>
       </footer>
      </div>
      <script src="https://vishaalagartha.github.io/assets/js/main.min.js">
      </script>
      <script>
       (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){ (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o), m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m) })(window,document,'script','//www.google-analytics.com/analytics.js','ga'); ga('create', '', 'auto'); ga('send', 'pageview');
      </script>
     </body>
    </html>
    

 
That was relatively easy!
## Static vs. Dynamic Content

The problem is that issuing a GET request to a static website returns all the
data the user will see. However, issuing a GET request to a **dynamic** website,
or a website that uses JavaScript to load content will not.

Consider the following url: https://www.basketball-
reference.com/players/j/jamesle01.html.

If we go to this page we can clearly see that **Per 36 Minutes** is clearly a
table on the page. But, this content is loaded dynamically. If we use the Chrome
inspector we can see this quite clearly:

![Stats Table](/assets/2020-01-16-bbref-scraper_1.png) 
 
The Per 36 Minutes table is in green, indicating that is loaded dynamically!
Moreover, let's try and find this table using a GET request: 

**In [4]:**

{% highlight python %}
r = get('https://www.basketball-reference.com/players/j/jamesle01.html')
if r.status_code==200:
    soup = BeautifulSoup(r.content, 'html.parser')
    # Find all tables on the website
    for table in soup.find_all('table'):
        # Print the 'id' attribute of the table
        print(table.attrs['id'])
{% endhighlight %}

    per_game

 
Evidently, the `per_minute` table is not captured. So what's the workaround? 
 
## Approach 1: Use a dynamic scraper

One approach is to use a dynamic scraper like [Selenium](https://selenium-
python.readthedocs.io/).

Selenium and other dynamic scrapers launch a WebDriver or an instance of your
browser which eventually loads ALL the content on the site. Then, you can
provide your scraping logic by forcing the scraper to wait for certain
selectors.

Another, excellent, but less used Python scraper is
[pyppeteer](https://github.com/miyakogi/pyppeteer).
I initially started off by using this as my dynamic scraper. Let's take a look
at how to do scrape this site using pyppeteer. 

**In [16]:**

{% highlight python %}
import pandas as pd
from pyppeteer import launch
from requests import get
from bs4 import BeautifulSoup
import asyncio
import nest_asyncio
nest_asyncio.apply()

async def get_player_selector(url, selector):
    # Launch the browser and a new page
    browser = await launch()
    page = await browser.newPage()
    await page.goto(url)
    await page.waitForSelector(f'{selector}')
    table = await page.querySelectorEval(f'{selector}', '(element) => element.outerHTML')
    await browser.close()
    # Use pandas to read the table easily
    return table
{% endhighlight %}

**In [17]:**

{% highlight python %}
url = 'https://www.basketball-reference.com/players/j/jamesle01.html'
selector = '#per_minute'

table = asyncio.get_event_loop().run_until_complete(get_player_selector(url, selector))
table
{% endhighlight %}




    '<table class="row_summable sortable stats_table now_sortable sliding_cols" id="per_minute" data-cols-to-freeze="1"><caption>Per 36 Minutes Table</caption>\n   <colgroup><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col><col></colgroup>\n   <thead>      \n      <tr>\n         <th aria-label="If listed as single number, the year the season ended.★ - Indicates All-Star for league.Only on regular season tables." data-stat="season" scope="col" class=" poptip sort_default_asc center" data-tip="If listed as single number, the year the season ended.<br>★ - Indicates All-Star for league.<br>Only on regular season tables.">Season</th>\n         <th aria-label="Player\'s age on February 1 of the season" data-stat="age" scope="col" class=" poptip sort_default_asc center" data-tip="Player\'s age on February 1 of the season">Age</th>\n         <th aria-label="Team" data-stat="team_id" scope="col" class=" poptip sort_default_asc center" data-tip="Team">Tm</th>\n         <th aria-label="League" data-stat="lg_id" scope="col" class=" poptip sort_default_asc center" data-tip="League">Lg</th>\n         <th aria-label="Position" data-stat="pos" scope="col" class=" poptip sort_default_asc center" data-tip="Position">Pos</th>\n         <th aria-label="Games" data-stat="g" scope="col" class=" poptip center" data-tip="Games">G</th>\n         <th aria-label="Games Started" data-stat="gs" scope="col" class=" poptip center" data-tip="Games Started">GS</th>\n         <th aria-label="Minutes Played" data-stat="mp" scope="col" class=" poptip center" data-tip="Minutes Played">MP</th>\n         <th aria-label="Field Goals Per 36 Minutes" data-stat="fg_per_mp" scope="col" class=" poptip center" data-tip="Field Goals Per 36 Minutes">FG</th>\n         <th aria-label="Field Goal Attempts Per 36 Minutes" data-stat="fga_per_mp" scope="col" class=" poptip center" data-tip="Field Goal Attempts Per 36 Minutes">FGA</th>\n         <th aria-label="Field Goal Percentage" data-stat="fg_pct" scope="col" class=" poptip center" data-tip="Field Goal Percentage">FG%</th>\n         <th aria-label="3-Point Field Goals Per 36 Minutes" data-stat="fg3_per_mp" scope="col" class=" poptip center" data-tip="3-Point Field Goals Per 36 Minutes">3P</th>\n         <th aria-label="3-Point Field Goal Attempts Per 36 Minutes" data-stat="fg3a_per_mp" scope="col" class=" poptip center" data-tip="3-Point Field Goal Attempts Per 36 Minutes">3PA</th>\n         <th aria-label="3-Point Field Goal Percentage" data-stat="fg3_pct" scope="col" class=" poptip center" data-tip="3-Point Field Goal Percentage">3P%</th>\n         <th aria-label="2-Point Field Goals Per 36 Minutes" data-stat="fg2_per_mp" scope="col" class=" poptip center" data-tip="2-Point Field Goals Per 36 Minutes">2P</th>\n         <th aria-label="2-Point Field Goal Attempts Per 36 Minutes" data-stat="fg2a_per_mp" scope="col" class=" poptip center" data-tip="2-Point Field Goal Attempts Per 36 Minutes">2PA</th>\n         <th aria-label="2-Point Field Goal Percentage" data-stat="fg2_pct" scope="col" class=" poptip center" data-tip="2-Point Field Goal Percentage">2P%</th>\n         <th aria-label="Free Throws Per 36 Minutes" data-stat="ft_per_mp" scope="col" class=" poptip center" data-tip="Free Throws Per 36 Minutes">FT</th>\n         <th aria-label="Free Throw Attempts Per 36 Minutes" data-stat="fta_per_mp" scope="col" class=" poptip center" data-tip="Free Throw Attempts Per 36 Minutes">FTA</th>\n         <th aria-label="Free Throw Percentage" data-stat="ft_pct" scope="col" class=" poptip center" data-tip="Free Throw Percentage">FT%</th>\n         <th aria-label="Offensive Rebounds Per 36 Minutes" data-stat="orb_per_mp" scope="col" class=" poptip center" data-tip="Offensive Rebounds Per 36 Minutes">ORB</th>\n         <th aria-label="Defensive Rebounds Per 36 Minutes" data-stat="drb_per_mp" scope="col" class=" poptip center" data-tip="Defensive Rebounds Per 36 Minutes">DRB</th>\n         <th aria-label="Total Rebounds Per 36 Minutes" data-stat="trb_per_mp" scope="col" class=" poptip center" data-tip="Total Rebounds Per 36 Minutes">TRB</th>\n         <th aria-label="Assists Per 36 Minutes" data-stat="ast_per_mp" scope="col" class=" poptip center" data-tip="Assists Per 36 Minutes">AST</th>\n         <th aria-label="Steals Per 36 Minutes" data-stat="stl_per_mp" scope="col" class=" poptip center" data-tip="Steals Per 36 Minutes">STL</th>\n         <th aria-label="Blocks Per 36 Minutes" data-stat="blk_per_mp" scope="col" class=" poptip center" data-tip="Blocks Per 36 Minutes">BLK</th>\n         <th aria-label="Turnovers Per 36 Minutes" data-stat="tov_per_mp" scope="col" class=" poptip center" data-tip="Turnovers Per 36 Minutes">TOV</th>\n         <th aria-label="Personal Fouls Per 36 Minutes" data-stat="pf_per_mp" scope="col" class=" poptip center" data-tip="Personal Fouls Per 36 Minutes">PF</th>\n         <th aria-label="Points Per 36 Minutes" data-stat="pts_per_mp" scope="col" class=" poptip center" data-tip="Points Per 36 Minutes">PTS</th>\n      </tr>\n      \n   </thead>\n   <tbody>\n<tr id="per_minute.2004" class="full_table" data-row="0"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2004/">2003-04</a></th><td class="center " data-stat="age">19</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2004.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2004.html">NBA</a></td><td class="center " data-stat="pos">SG</td><td class="right " data-stat="g">79</td><td class="right " data-stat="gs">79</td><td class="right " data-stat="mp">3122</td><td class="right " data-stat="fg_per_mp">7.2</td><td class="right " data-stat="fga_per_mp">17.2</td><td class="right " data-stat="fg_pct">.417</td><td class="right " data-stat="fg3_per_mp">0.7</td><td class="right " data-stat="fg3a_per_mp">2.5</td><td class="right " data-stat="fg3_pct">.290</td><td class="right " data-stat="fg2_per_mp">6.4</td><td class="right " data-stat="fg2a_per_mp">14.7</td><td class="right " data-stat="fg2_pct">.438</td><td class="right " data-stat="ft_per_mp">4.0</td><td class="right " data-stat="fta_per_mp">5.3</td><td class="right " data-stat="ft_pct">.754</td><td class="right " data-stat="orb_per_mp">1.1</td><td class="right " data-stat="drb_per_mp">3.8</td><td class="right " data-stat="trb_per_mp">5.0</td><td class="right " data-stat="ast_per_mp">5.4</td><td class="right " data-stat="stl_per_mp">1.5</td><td class="right " data-stat="blk_per_mp">0.7</td><td class="right " data-stat="tov_per_mp">3.1</td><td class="right " data-stat="pf_per_mp">1.7</td><td class="right " data-stat="pts_per_mp">19.1</td></tr>\n<tr id="per_minute.2005" class="full_table" data-row="1"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2005/">2004-05</a><span class="sr_star"></span></th><td class="center " data-stat="age">20</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2005.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2005.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">80</td><td class="right " data-stat="gs">80</td><td class="right " data-stat="mp"><strong>3388</strong></td><td class="right " data-stat="fg_per_mp">8.4</td><td class="right " data-stat="fga_per_mp">17.9</td><td class="right " data-stat="fg_pct">.472</td><td class="right " data-stat="fg3_per_mp">1.1</td><td class="right " data-stat="fg3a_per_mp">3.3</td><td class="right " data-stat="fg3_pct">.351</td><td class="right " data-stat="fg2_per_mp">7.3</td><td class="right " data-stat="fg2a_per_mp">14.6</td><td class="right " data-stat="fg2_pct">.499</td><td class="right " data-stat="ft_per_mp">5.1</td><td class="right " data-stat="fta_per_mp">6.8</td><td class="right " data-stat="ft_pct">.750</td><td class="right " data-stat="orb_per_mp">1.2</td><td class="right " data-stat="drb_per_mp">5.1</td><td class="right " data-stat="trb_per_mp">6.2</td><td class="right " data-stat="ast_per_mp">6.1</td><td class="right " data-stat="stl_per_mp">1.9</td><td class="right " data-stat="blk_per_mp">0.6</td><td class="right " data-stat="tov_per_mp">2.8</td><td class="right " data-stat="pf_per_mp">1.6</td><td class="right " data-stat="pts_per_mp">23.1</td></tr>\n<tr id="per_minute.2006" class="full_table" data-row="2"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2006/">2005-06</a><span class="sr_star"></span></th><td class="center " data-stat="age">21</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2006.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2006.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">79</td><td class="right " data-stat="gs">79</td><td class="right " data-stat="mp">3361</td><td class="right " data-stat="fg_per_mp">9.4</td><td class="right " data-stat="fga_per_mp">19.5</td><td class="right " data-stat="fg_pct">.480</td><td class="right " data-stat="fg3_per_mp">1.4</td><td class="right " data-stat="fg3a_per_mp">4.1</td><td class="right " data-stat="fg3_pct">.335</td><td class="right " data-stat="fg2_per_mp">8.0</td><td class="right " data-stat="fg2a_per_mp">15.5</td><td class="right " data-stat="fg2_pct">.518</td><td class="right " data-stat="ft_per_mp">6.4</td><td class="right " data-stat="fta_per_mp">8.7</td><td class="right " data-stat="ft_pct">.738</td><td class="right " data-stat="orb_per_mp">0.8</td><td class="right " data-stat="drb_per_mp">5.2</td><td class="right " data-stat="trb_per_mp">6.0</td><td class="right " data-stat="ast_per_mp">5.6</td><td class="right " data-stat="stl_per_mp">1.3</td><td class="right " data-stat="blk_per_mp">0.7</td><td class="right " data-stat="tov_per_mp">2.8</td><td class="right " data-stat="pf_per_mp">1.9</td><td class="right " data-stat="pts_per_mp">26.5</td></tr>\n<tr id="per_minute.2007" class="full_table" data-row="3"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2007/">2006-07</a><span class="sr_star"></span></th><td class="center " data-stat="age">22</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2007.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2007.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">78</td><td class="right " data-stat="gs">78</td><td class="right " data-stat="mp"><strong>3190</strong></td><td class="right " data-stat="fg_per_mp">8.7</td><td class="right " data-stat="fga_per_mp">18.3</td><td class="right " data-stat="fg_pct">.476</td><td class="right " data-stat="fg3_per_mp">1.1</td><td class="right " data-stat="fg3a_per_mp">3.5</td><td class="right " data-stat="fg3_pct">.319</td><td class="right " data-stat="fg2_per_mp">7.6</td><td class="right " data-stat="fg2a_per_mp">14.8</td><td class="right " data-stat="fg2_pct">.513</td><td class="right " data-stat="ft_per_mp">5.5</td><td class="right " data-stat="fta_per_mp">7.9</td><td class="right " data-stat="ft_pct">.698</td><td class="right " data-stat="orb_per_mp">0.9</td><td class="right " data-stat="drb_per_mp">5.0</td><td class="right " data-stat="trb_per_mp">5.9</td><td class="right " data-stat="ast_per_mp">5.3</td><td class="right " data-stat="stl_per_mp">1.4</td><td class="right " data-stat="blk_per_mp">0.6</td><td class="right " data-stat="tov_per_mp">2.8</td><td class="right " data-stat="pf_per_mp">1.9</td><td class="right " data-stat="pts_per_mp">24.1</td></tr>\n<tr id="per_minute.2008" class="full_table" data-row="4"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2008/">2007-08</a><span class="sr_star"></span></th><td class="center " data-stat="age">23</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2008.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2008.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">75</td><td class="right " data-stat="gs">74</td><td class="right " data-stat="mp">3027</td><td class="right " data-stat="fg_per_mp">9.4</td><td class="right " data-stat="fga_per_mp">19.5</td><td class="right " data-stat="fg_pct">.484</td><td class="right " data-stat="fg3_per_mp">1.3</td><td class="right " data-stat="fg3a_per_mp">4.3</td><td class="right " data-stat="fg3_pct">.315</td><td class="right " data-stat="fg2_per_mp">8.1</td><td class="right " data-stat="fg2a_per_mp">15.3</td><td class="right " data-stat="fg2_pct">.531</td><td class="right " data-stat="ft_per_mp">6.5</td><td class="right " data-stat="fta_per_mp">9.2</td><td class="right " data-stat="ft_pct">.712</td><td class="right " data-stat="orb_per_mp">1.6</td><td class="right " data-stat="drb_per_mp">5.5</td><td class="right " data-stat="trb_per_mp">7.0</td><td class="right " data-stat="ast_per_mp">6.4</td><td class="right " data-stat="stl_per_mp">1.6</td><td class="right " data-stat="blk_per_mp">1.0</td><td class="right " data-stat="tov_per_mp">3.0</td><td class="right " data-stat="pf_per_mp">2.0</td><td class="right " data-stat="pts_per_mp">26.8</td></tr>\n<tr id="per_minute.2009" class="full_table" data-row="5"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2009/">2008-09</a><span class="sr_star"></span></th><td class="center " data-stat="age">24</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2009.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2009.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">81</td><td class="right " data-stat="gs">81</td><td class="right " data-stat="mp">3054</td><td class="right " data-stat="fg_per_mp">9.3</td><td class="right " data-stat="fga_per_mp">19.0</td><td class="right " data-stat="fg_pct">.489</td><td class="right " data-stat="fg3_per_mp">1.6</td><td class="right " data-stat="fg3a_per_mp">4.5</td><td class="right " data-stat="fg3_pct">.344</td><td class="right " data-stat="fg2_per_mp">7.7</td><td class="right " data-stat="fg2a_per_mp">14.5</td><td class="right " data-stat="fg2_pct">.535</td><td class="right " data-stat="ft_per_mp">7.0</td><td class="right " data-stat="fta_per_mp">9.0</td><td class="right " data-stat="ft_pct">.780</td><td class="right " data-stat="orb_per_mp">1.2</td><td class="right " data-stat="drb_per_mp">6.0</td><td class="right " data-stat="trb_per_mp">7.2</td><td class="right " data-stat="ast_per_mp">6.9</td><td class="right " data-stat="stl_per_mp">1.6</td><td class="right " data-stat="blk_per_mp">1.1</td><td class="right " data-stat="tov_per_mp">2.8</td><td class="right " data-stat="pf_per_mp">1.6</td><td class="right " data-stat="pts_per_mp">27.2</td></tr>\n<tr id="per_minute.2010" class="full_table" data-row="6"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2010/">2009-10</a><span class="sr_star"></span></th><td class="center " data-stat="age">25</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2010.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2010.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">76</td><td class="right " data-stat="gs">76</td><td class="right " data-stat="mp">2966</td><td class="right " data-stat="fg_per_mp">9.3</td><td class="right " data-stat="fga_per_mp">18.5</td><td class="right " data-stat="fg_pct">.503</td><td class="right " data-stat="fg3_per_mp">1.6</td><td class="right " data-stat="fg3a_per_mp">4.7</td><td class="right " data-stat="fg3_pct">.333</td><td class="right " data-stat="fg2_per_mp">7.8</td><td class="right " data-stat="fg2a_per_mp">13.8</td><td class="right " data-stat="fg2_pct">.560</td><td class="right " data-stat="ft_per_mp">7.2</td><td class="right " data-stat="fta_per_mp">9.4</td><td class="right " data-stat="ft_pct">.767</td><td class="right " data-stat="orb_per_mp">0.9</td><td class="right " data-stat="drb_per_mp">5.9</td><td class="right " data-stat="trb_per_mp">6.7</td><td class="right " data-stat="ast_per_mp">7.9</td><td class="right " data-stat="stl_per_mp">1.5</td><td class="right " data-stat="blk_per_mp">0.9</td><td class="right " data-stat="tov_per_mp">3.2</td><td class="right " data-stat="pf_per_mp">1.4</td><td class="right " data-stat="pts_per_mp">27.4</td></tr>\n<tr id="per_minute.2011" class="full_table" data-row="7"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2011/">2010-11</a><span class="sr_star"></span></th><td class="center " data-stat="age">26</td><td class="left " data-stat="team_id"><a href="/teams/MIA/2011.html">MIA</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2011.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">79</td><td class="right " data-stat="gs">79</td><td class="right " data-stat="mp">3063</td><td class="right " data-stat="fg_per_mp">8.9</td><td class="right " data-stat="fga_per_mp">17.5</td><td class="right " data-stat="fg_pct">.510</td><td class="right " data-stat="fg3_per_mp">1.1</td><td class="right " data-stat="fg3a_per_mp">3.3</td><td class="right " data-stat="fg3_pct">.330</td><td class="right " data-stat="fg2_per_mp">7.8</td><td class="right " data-stat="fg2a_per_mp">14.2</td><td class="right " data-stat="fg2_pct">.552</td><td class="right " data-stat="ft_per_mp">5.9</td><td class="right " data-stat="fta_per_mp">7.8</td><td class="right " data-stat="ft_pct">.759</td><td class="right " data-stat="orb_per_mp">0.9</td><td class="right " data-stat="drb_per_mp">6.0</td><td class="right " data-stat="trb_per_mp">6.9</td><td class="right " data-stat="ast_per_mp">6.5</td><td class="right " data-stat="stl_per_mp">1.5</td><td class="right " data-stat="blk_per_mp">0.6</td><td class="right " data-stat="tov_per_mp">3.3</td><td class="right " data-stat="pf_per_mp">1.9</td><td class="right " data-stat="pts_per_mp">24.8</td></tr>\n<tr id="per_minute.2012" class="full_table" data-row="8"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2012/">2011-12</a><span class="sr_star"></span></th><td class="center " data-stat="age">27</td><td class="left " data-stat="team_id"><a href="/teams/MIA/2012.html">MIA</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2012.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">62</td><td class="right " data-stat="gs">62</td><td class="right " data-stat="mp">2326</td><td class="right " data-stat="fg_per_mp">9.6</td><td class="right " data-stat="fga_per_mp">18.1</td><td class="right " data-stat="fg_pct">.531</td><td class="right " data-stat="fg3_per_mp">0.8</td><td class="right " data-stat="fg3a_per_mp">2.3</td><td class="right " data-stat="fg3_pct">.362</td><td class="right " data-stat="fg2_per_mp">8.8</td><td class="right " data-stat="fg2a_per_mp">15.8</td><td class="right " data-stat="fg2_pct">.556</td><td class="right " data-stat="ft_per_mp">6.0</td><td class="right " data-stat="fta_per_mp">7.8</td><td class="right " data-stat="ft_pct">.771</td><td class="right " data-stat="orb_per_mp">1.5</td><td class="right " data-stat="drb_per_mp">6.2</td><td class="right " data-stat="trb_per_mp">7.6</td><td class="right " data-stat="ast_per_mp">6.0</td><td class="right " data-stat="stl_per_mp">1.8</td><td class="right " data-stat="blk_per_mp">0.8</td><td class="right " data-stat="tov_per_mp">3.3</td><td class="right " data-stat="pf_per_mp">1.5</td><td class="right " data-stat="pts_per_mp">26.0</td></tr>\n<tr id="per_minute.2013" class="full_table" data-row="9"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2013/">2012-13</a><span class="sr_star"></span></th><td class="center " data-stat="age">28</td><td class="left " data-stat="team_id"><a href="/teams/MIA/2013.html">MIA</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2013.html">NBA</a></td><td class="center " data-stat="pos">PF</td><td class="right " data-stat="g">76</td><td class="right " data-stat="gs">76</td><td class="right " data-stat="mp">2877</td><td class="right " data-stat="fg_per_mp">9.6</td><td class="right " data-stat="fga_per_mp">16.9</td><td class="right " data-stat="fg_pct">.565</td><td class="right " data-stat="fg3_per_mp">1.3</td><td class="right " data-stat="fg3a_per_mp">3.2</td><td class="right " data-stat="fg3_pct">.406</td><td class="right " data-stat="fg2_per_mp">8.3</td><td class="right " data-stat="fg2a_per_mp">13.8</td><td class="right " data-stat="fg2_pct">.602</td><td class="right " data-stat="ft_per_mp">5.0</td><td class="right " data-stat="fta_per_mp">6.7</td><td class="right " data-stat="ft_pct">.753</td><td class="right " data-stat="orb_per_mp">1.2</td><td class="right " data-stat="drb_per_mp">6.4</td><td class="right " data-stat="trb_per_mp">7.6</td><td class="right " data-stat="ast_per_mp">6.9</td><td class="right " data-stat="stl_per_mp">1.6</td><td class="right " data-stat="blk_per_mp">0.8</td><td class="right " data-stat="tov_per_mp">2.8</td><td class="right " data-stat="pf_per_mp">1.4</td><td class="right " data-stat="pts_per_mp">25.5</td></tr>\n<tr id="per_minute.2014" class="full_table" data-row="10"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2014/">2013-14</a><span class="sr_star"></span></th><td class="center " data-stat="age">29</td><td class="left " data-stat="team_id"><a href="/teams/MIA/2014.html">MIA</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2014.html">NBA</a></td><td class="center " data-stat="pos">PF</td><td class="right " data-stat="g">77</td><td class="right " data-stat="gs">77</td><td class="right " data-stat="mp">2902</td><td class="right " data-stat="fg_per_mp">9.5</td><td class="right " data-stat="fga_per_mp">16.8</td><td class="right " data-stat="fg_pct">.567</td><td class="right " data-stat="fg3_per_mp">1.4</td><td class="right " data-stat="fg3a_per_mp">3.8</td><td class="right " data-stat="fg3_pct">.379</td><td class="right " data-stat="fg2_per_mp">8.1</td><td class="right " data-stat="fg2a_per_mp">13.0</td><td class="right " data-stat="fg2_pct">.622</td><td class="right " data-stat="ft_per_mp">5.4</td><td class="right " data-stat="fta_per_mp">7.3</td><td class="right " data-stat="ft_pct">.750</td><td class="right " data-stat="orb_per_mp">1.0</td><td class="right " data-stat="drb_per_mp">5.6</td><td class="right " data-stat="trb_per_mp">6.6</td><td class="right " data-stat="ast_per_mp">6.1</td><td class="right " data-stat="stl_per_mp">1.5</td><td class="right " data-stat="blk_per_mp">0.3</td><td class="right " data-stat="tov_per_mp">3.3</td><td class="right " data-stat="pf_per_mp">1.6</td><td class="right " data-stat="pts_per_mp">25.9</td></tr>\n<tr id="per_minute.2015" class="full_table" data-row="11"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2015/">2014-15</a><span class="sr_star"></span></th><td class="center " data-stat="age">30</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2015.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2015.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">69</td><td class="right " data-stat="gs">69</td><td class="right " data-stat="mp">2493</td><td class="right " data-stat="fg_per_mp">9.0</td><td class="right " data-stat="fga_per_mp">18.5</td><td class="right " data-stat="fg_pct">.488</td><td class="right " data-stat="fg3_per_mp">1.7</td><td class="right " data-stat="fg3a_per_mp">4.9</td><td class="right " data-stat="fg3_pct">.354</td><td class="right " data-stat="fg2_per_mp">7.3</td><td class="right " data-stat="fg2a_per_mp">13.6</td><td class="right " data-stat="fg2_pct">.536</td><td class="right " data-stat="ft_per_mp">5.4</td><td class="right " data-stat="fta_per_mp">7.6</td><td class="right " data-stat="ft_pct">.710</td><td class="right " data-stat="orb_per_mp">0.7</td><td class="right " data-stat="drb_per_mp">5.3</td><td class="right " data-stat="trb_per_mp">6.0</td><td class="right " data-stat="ast_per_mp">7.4</td><td class="right " data-stat="stl_per_mp">1.6</td><td class="right " data-stat="blk_per_mp">0.7</td><td class="right " data-stat="tov_per_mp">3.9</td><td class="right " data-stat="pf_per_mp">1.9</td><td class="right " data-stat="pts_per_mp">25.2</td></tr>\n<tr id="per_minute.2016" class="full_table" data-row="12"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2016/">2015-16</a><span class="sr_star"></span></th><td class="center " data-stat="age">31</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2016.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2016.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">76</td><td class="right " data-stat="gs">76</td><td class="right " data-stat="mp">2709</td><td class="right " data-stat="fg_per_mp">9.8</td><td class="right " data-stat="fga_per_mp">18.8</td><td class="right " data-stat="fg_pct">.520</td><td class="right " data-stat="fg3_per_mp">1.2</td><td class="right " data-stat="fg3a_per_mp">3.7</td><td class="right " data-stat="fg3_pct">.309</td><td class="right " data-stat="fg2_per_mp">8.6</td><td class="right " data-stat="fg2a_per_mp">15.1</td><td class="right " data-stat="fg2_pct">.573</td><td class="right " data-stat="ft_per_mp">4.8</td><td class="right " data-stat="fta_per_mp">6.5</td><td class="right " data-stat="ft_pct">.731</td><td class="right " data-stat="orb_per_mp">1.5</td><td class="right " data-stat="drb_per_mp">6.0</td><td class="right " data-stat="trb_per_mp">7.5</td><td class="right " data-stat="ast_per_mp">6.8</td><td class="right " data-stat="stl_per_mp">1.4</td><td class="right " data-stat="blk_per_mp">0.7</td><td class="right " data-stat="tov_per_mp">3.3</td><td class="right " data-stat="pf_per_mp">1.9</td><td class="right " data-stat="pts_per_mp">25.5</td></tr>\n<tr id="per_minute.2017" class="full_table" data-row="13"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2017/">2016-17</a><span class="sr_star"></span></th><td class="center " data-stat="age">32</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2017.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2017.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">74</td><td class="right " data-stat="gs">74</td><td class="right " data-stat="mp">2794</td><td class="right " data-stat="fg_per_mp">9.5</td><td class="right " data-stat="fga_per_mp">17.3</td><td class="right " data-stat="fg_pct">.548</td><td class="right " data-stat="fg3_per_mp">1.6</td><td class="right " data-stat="fg3a_per_mp">4.4</td><td class="right " data-stat="fg3_pct">.363</td><td class="right " data-stat="fg2_per_mp">7.9</td><td class="right " data-stat="fg2a_per_mp">12.9</td><td class="right " data-stat="fg2_pct">.611</td><td class="right " data-stat="ft_per_mp">4.6</td><td class="right " data-stat="fta_per_mp">6.8</td><td class="right " data-stat="ft_pct">.674</td><td class="right " data-stat="orb_per_mp">1.2</td><td class="right " data-stat="drb_per_mp">7.0</td><td class="right " data-stat="trb_per_mp">8.2</td><td class="right " data-stat="ast_per_mp">8.3</td><td class="right " data-stat="stl_per_mp">1.2</td><td class="right " data-stat="blk_per_mp">0.6</td><td class="right " data-stat="tov_per_mp">3.9</td><td class="right " data-stat="pf_per_mp">1.7</td><td class="right " data-stat="pts_per_mp">25.2</td></tr>\n<tr id="per_minute.2018" class="full_table" data-row="14"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2018/">2017-18</a><span class="sr_star"></span></th><td class="center " data-stat="age">33</td><td class="left " data-stat="team_id"><a href="/teams/CLE/2018.html">CLE</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2018.html">NBA</a></td><td class="center " data-stat="pos">PF</td><td class="right " data-stat="g"><strong>82</strong></td><td class="right " data-stat="gs">82</td><td class="right " data-stat="mp"><strong>3026</strong></td><td class="right " data-stat="fg_per_mp">10.2</td><td class="right " data-stat="fga_per_mp">18.8</td><td class="right " data-stat="fg_pct">.542</td><td class="right " data-stat="fg3_per_mp">1.8</td><td class="right " data-stat="fg3a_per_mp">4.8</td><td class="right " data-stat="fg3_pct">.367</td><td class="right " data-stat="fg2_per_mp">8.4</td><td class="right " data-stat="fg2a_per_mp">14.0</td><td class="right " data-stat="fg2_pct">.603</td><td class="right " data-stat="ft_per_mp">4.6</td><td class="right " data-stat="fta_per_mp">6.3</td><td class="right " data-stat="ft_pct">.731</td><td class="right " data-stat="orb_per_mp">1.2</td><td class="right " data-stat="drb_per_mp">7.3</td><td class="right " data-stat="trb_per_mp">8.4</td><td class="right " data-stat="ast_per_mp">8.9</td><td class="right " data-stat="stl_per_mp">1.4</td><td class="right " data-stat="blk_per_mp">0.8</td><td class="right " data-stat="tov_per_mp">4.1</td><td class="right " data-stat="pf_per_mp">1.6</td><td class="right " data-stat="pts_per_mp">26.8</td></tr>\n<tr id="per_minute.2019" class="full_table" data-row="15"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2019/">2018-19</a><span class="sr_star"></span></th><td class="center " data-stat="age">34</td><td class="left " data-stat="team_id"><a href="/teams/LAL/2019.html">LAL</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2019.html">NBA</a></td><td class="center " data-stat="pos">SF</td><td class="right " data-stat="g">55</td><td class="right " data-stat="gs">55</td><td class="right " data-stat="mp">1937</td><td class="right " data-stat="fg_per_mp">10.4</td><td class="right " data-stat="fga_per_mp">20.4</td><td class="right " data-stat="fg_pct">.510</td><td class="right " data-stat="fg3_per_mp">2.1</td><td class="right " data-stat="fg3a_per_mp">6.1</td><td class="right " data-stat="fg3_pct">.339</td><td class="right " data-stat="fg2_per_mp">8.3</td><td class="right " data-stat="fg2a_per_mp">14.3</td><td class="right " data-stat="fg2_pct">.582</td><td class="right " data-stat="ft_per_mp">5.2</td><td class="right " data-stat="fta_per_mp">7.8</td><td class="right " data-stat="ft_pct">.665</td><td class="right " data-stat="orb_per_mp">1.1</td><td class="right " data-stat="drb_per_mp">7.6</td><td class="right " data-stat="trb_per_mp">8.6</td><td class="right " data-stat="ast_per_mp">8.4</td><td class="right " data-stat="stl_per_mp">1.3</td><td class="right " data-stat="blk_per_mp">0.6</td><td class="right " data-stat="tov_per_mp">3.7</td><td class="right " data-stat="pf_per_mp">1.7</td><td class="right " data-stat="pts_per_mp">28.0</td></tr>\n<tr id="per_minute.2020" class="full_table" data-row="16"><th scope="row" class="left " data-stat="season"><a href="/players/j/jamesle01/gamelog/2020/">2019-20</a></th><td class="center " data-stat="age">35</td><td class="left " data-stat="team_id"><a href="/teams/LAL/2020.html">LAL</a></td><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2020.html">NBA</a></td><td class="center " data-stat="pos">PG</td><td class="right " data-stat="g">39</td><td class="right " data-stat="gs">39</td><td class="right " data-stat="mp">1362</td><td class="right " data-stat="fg_per_mp">10.0</td><td class="right " data-stat="fga_per_mp">20.4</td><td class="right " data-stat="fg_pct">.491</td><td class="right " data-stat="fg3_per_mp">2.2</td><td class="right " data-stat="fg3a_per_mp">6.3</td><td class="right " data-stat="fg3_pct">.346</td><td class="right " data-stat="fg2_per_mp">7.8</td><td class="right " data-stat="fg2a_per_mp">14.0</td><td class="right " data-stat="fg2_pct">.557</td><td class="right " data-stat="ft_per_mp">4.0</td><td class="right " data-stat="fta_per_mp">5.7</td><td class="right " data-stat="ft_pct">.694</td><td class="right " data-stat="orb_per_mp">1.1</td><td class="right " data-stat="drb_per_mp">6.9</td><td class="right " data-stat="trb_per_mp">8.0</td><td class="right " data-stat="ast_per_mp">11.2</td><td class="right " data-stat="stl_per_mp">1.3</td><td class="right " data-stat="blk_per_mp">0.5</td><td class="right " data-stat="tov_per_mp">3.9</td><td class="right " data-stat="pf_per_mp">1.8</td><td class="right " data-stat="pts_per_mp">26.1</td></tr>\n\n   </tbody>\n   <tfoot><tr data-row="17"><th scope="row" class="left " data-stat="season">Career</th><td class="center iz" data-stat="age"></td><td class="left iz" data-stat="team_id"></td><td class="left " data-stat="lg_id">NBA</td><td class="center iz" data-stat="pos"></td><td class="right " data-stat="g">1237</td><td class="right " data-stat="gs">1236</td><td class="right " data-stat="mp">47597</td><td class="right " data-stat="fg_per_mp">9.2</td><td class="right " data-stat="fga_per_mp">18.3</td><td class="right " data-stat="fg_pct">.504</td><td class="right " data-stat="fg3_per_mp">1.4</td><td class="right " data-stat="fg3a_per_mp">4.0</td><td class="right " data-stat="fg3_pct">.344</td><td class="right " data-stat="fg2_per_mp">7.9</td><td class="right " data-stat="fg2a_per_mp">14.4</td><td class="right " data-stat="fg2_pct">.548</td><td class="right " data-stat="ft_per_mp">5.5</td><td class="right " data-stat="fta_per_mp">7.5</td><td class="right " data-stat="ft_pct">.735</td><td class="right " data-stat="orb_per_mp">1.1</td><td class="right " data-stat="drb_per_mp">5.8</td><td class="right " data-stat="trb_per_mp">6.9</td><td class="right " data-stat="ast_per_mp">6.9</td><td class="right " data-stat="stl_per_mp">1.5</td><td class="right " data-stat="blk_per_mp">0.7</td><td class="right " data-stat="tov_per_mp">3.3</td><td class="right " data-stat="pf_per_mp">1.7</td><td class="right " data-stat="pts_per_mp">25.4</td></tr>\n<tr class="blank_table partial_table" data-row="18"><th scope="row" class="left iz" data-stat="season"></th><td class="center iz" data-stat="age"></td><td class="left iz" data-stat="team_id"></td><td class="left iz" data-stat="lg_id"></td><td class="center iz" data-stat="pos"></td><td class="right iz" data-stat="g"></td><td class="right iz" data-stat="gs"></td><td class="right iz" data-stat="mp"></td><td class="right iz" data-stat="fg_per_mp"></td><td class="right iz" data-stat="fga_per_mp"></td><td class="right iz" data-stat="fg_pct"></td><td class="right iz" data-stat="fg3_per_mp"></td><td class="right iz" data-stat="fg3a_per_mp"></td><td class="right iz" data-stat="fg3_pct"></td><td class="right iz" data-stat="fg2_per_mp"></td><td class="right iz" data-stat="fg2a_per_mp"></td><td class="right iz" data-stat="fg2_pct"></td><td class="right iz" data-stat="ft_per_mp"></td><td class="right iz" data-stat="fta_per_mp"></td><td class="right iz" data-stat="ft_pct"></td><td class="right iz" data-stat="orb_per_mp"></td><td class="right iz" data-stat="drb_per_mp"></td><td class="right iz" data-stat="trb_per_mp"></td><td class="right iz" data-stat="ast_per_mp"></td><td class="right iz" data-stat="stl_per_mp"></td><td class="right iz" data-stat="blk_per_mp"></td><td class="right iz" data-stat="tov_per_mp"></td><td class="right iz" data-stat="pf_per_mp"></td><td class="right iz" data-stat="pts_per_mp"></td></tr>\n<tr data-row="19"><th scope="row" class="left " data-stat="season">11 seasons</th><td class="center iz" data-stat="age"></td><td class="left " data-stat="team_id"><a href="/teams/CLE/">CLE</a></td><td class="left " data-stat="lg_id">NBA</td><td class="center iz" data-stat="pos"></td><td class="right " data-stat="g">849</td><td class="right " data-stat="gs">848</td><td class="right " data-stat="mp">33130</td><td class="right " data-stat="fg_per_mp">9.1</td><td class="right " data-stat="fga_per_mp">18.5</td><td class="right " data-stat="fg_pct">.492</td><td class="right " data-stat="fg3_per_mp">1.4</td><td class="right " data-stat="fg3a_per_mp">4.0</td><td class="right " data-stat="fg3_pct">.337</td><td class="right " data-stat="fg2_per_mp">7.7</td><td class="right " data-stat="fg2a_per_mp">14.5</td><td class="right " data-stat="fg2_pct">.535</td><td class="right " data-stat="ft_per_mp">5.6</td><td class="right " data-stat="fta_per_mp">7.6</td><td class="right " data-stat="ft_pct">.733</td><td class="right " data-stat="orb_per_mp">1.1</td><td class="right " data-stat="drb_per_mp">5.6</td><td class="right " data-stat="trb_per_mp">6.7</td><td class="right " data-stat="ast_per_mp">6.8</td><td class="right " data-stat="stl_per_mp">1.5</td><td class="right " data-stat="blk_per_mp">0.8</td><td class="right " data-stat="tov_per_mp">3.2</td><td class="right " data-stat="pf_per_mp">1.8</td><td class="right " data-stat="pts_per_mp">25.1</td></tr>\n<tr data-row="20"><th scope="row" class="left " data-stat="season">4 seasons</th><td class="center iz" data-stat="age"></td><td class="left " data-stat="team_id"><a href="/teams/MIA/">MIA</a></td><td class="left " data-stat="lg_id">NBA</td><td class="center iz" data-stat="pos"></td><td class="right " data-stat="g">294</td><td class="right " data-stat="gs">294</td><td class="right " data-stat="mp">11168</td><td class="right " data-stat="fg_per_mp">9.4</td><td class="right " data-stat="fga_per_mp">17.3</td><td class="right " data-stat="fg_pct">.543</td><td class="right " data-stat="fg3_per_mp">1.2</td><td class="right " data-stat="fg3a_per_mp">3.2</td><td class="right " data-stat="fg3_pct">.369</td><td class="right " data-stat="fg2_per_mp">8.2</td><td class="right " data-stat="fg2a_per_mp">14.1</td><td class="right " data-stat="fg2_pct">.582</td><td class="right " data-stat="ft_per_mp">5.6</td><td class="right " data-stat="fta_per_mp">7.4</td><td class="right " data-stat="ft_pct">.758</td><td class="right " data-stat="orb_per_mp">1.1</td><td class="right " data-stat="drb_per_mp">6.0</td><td class="right " data-stat="trb_per_mp">7.2</td><td class="right " data-stat="ast_per_mp">6.4</td><td class="right " data-stat="stl_per_mp">1.6</td><td class="right " data-stat="blk_per_mp">0.6</td><td class="right " data-stat="tov_per_mp">3.2</td><td class="right " data-stat="pf_per_mp">1.6</td><td class="right " data-stat="pts_per_mp">25.5</td></tr>\n<tr data-row="21"><th scope="row" class="left " data-stat="season">2 seasons</th><td class="center iz" data-stat="age"></td><td class="left " data-stat="team_id"><a href="/teams/LAL/">LAL</a></td><td class="left " data-stat="lg_id">NBA</td><td class="center iz" data-stat="pos"></td><td class="right " data-stat="g">94</td><td class="right " data-stat="gs">94</td><td class="right " data-stat="mp">3299</td><td class="right " data-stat="fg_per_mp">10.2</td><td class="right " data-stat="fga_per_mp">20.4</td><td class="right " data-stat="fg_pct">.502</td><td class="right " data-stat="fg3_per_mp">2.1</td><td class="right " data-stat="fg3a_per_mp">6.2</td><td class="right " data-stat="fg3_pct">.342</td><td class="right " data-stat="fg2_per_mp">8.1</td><td class="right " data-stat="fg2a_per_mp">14.2</td><td class="right " data-stat="fg2_pct">.572</td><td class="right " data-stat="ft_per_mp">4.7</td><td class="right " data-stat="fta_per_mp">6.9</td><td class="right " data-stat="ft_pct">.675</td><td class="right " data-stat="orb_per_mp">1.1</td><td class="right " data-stat="drb_per_mp">7.3</td><td class="right " data-stat="trb_per_mp">8.4</td><td class="right " data-stat="ast_per_mp">9.6</td><td class="right " data-stat="stl_per_mp">1.3</td><td class="right " data-stat="blk_per_mp">0.6</td><td class="right " data-stat="tov_per_mp">3.7</td><td class="right " data-stat="pf_per_mp">1.8</td><td class="right " data-stat="pts_per_mp">27.2</td></tr>\n\n   </tfoot>\n\n</table>'


 
Aha! Now that get's the relevant content! We can also use the Pandas library to
read this html table relatively easily: 

**In [18]:**

{% highlight python %}
df = pd.read_html(table)[0]
df
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
      <th>Season</th>
      <th>Age</th>
      <th>Tm</th>
      <th>Lg</th>
      <th>Pos</th>
      <th>G</th>
      <th>GS</th>
      <th>MP</th>
      <th>FG</th>
      <th>FGA</th>
      <th>...</th>
      <th>FT%</th>
      <th>ORB</th>
      <th>DRB</th>
      <th>TRB</th>
      <th>AST</th>
      <th>STL</th>
      <th>BLK</th>
      <th>TOV</th>
      <th>PF</th>
      <th>PTS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2003-04</td>
      <td>19.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SG</td>
      <td>79.0</td>
      <td>79.0</td>
      <td>3122.0</td>
      <td>7.2</td>
      <td>17.2</td>
      <td>...</td>
      <td>0.754</td>
      <td>1.1</td>
      <td>3.8</td>
      <td>5.0</td>
      <td>5.4</td>
      <td>1.5</td>
      <td>0.7</td>
      <td>3.1</td>
      <td>1.7</td>
      <td>19.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2004-05</td>
      <td>20.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>80.0</td>
      <td>80.0</td>
      <td>3388.0</td>
      <td>8.4</td>
      <td>17.9</td>
      <td>...</td>
      <td>0.750</td>
      <td>1.2</td>
      <td>5.1</td>
      <td>6.2</td>
      <td>6.1</td>
      <td>1.9</td>
      <td>0.6</td>
      <td>2.8</td>
      <td>1.6</td>
      <td>23.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005-06</td>
      <td>21.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>79.0</td>
      <td>79.0</td>
      <td>3361.0</td>
      <td>9.4</td>
      <td>19.5</td>
      <td>...</td>
      <td>0.738</td>
      <td>0.8</td>
      <td>5.2</td>
      <td>6.0</td>
      <td>5.6</td>
      <td>1.3</td>
      <td>0.7</td>
      <td>2.8</td>
      <td>1.9</td>
      <td>26.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2006-07</td>
      <td>22.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>78.0</td>
      <td>78.0</td>
      <td>3190.0</td>
      <td>8.7</td>
      <td>18.3</td>
      <td>...</td>
      <td>0.698</td>
      <td>0.9</td>
      <td>5.0</td>
      <td>5.9</td>
      <td>5.3</td>
      <td>1.4</td>
      <td>0.6</td>
      <td>2.8</td>
      <td>1.9</td>
      <td>24.1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2007-08</td>
      <td>23.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>75.0</td>
      <td>74.0</td>
      <td>3027.0</td>
      <td>9.4</td>
      <td>19.5</td>
      <td>...</td>
      <td>0.712</td>
      <td>1.6</td>
      <td>5.5</td>
      <td>7.0</td>
      <td>6.4</td>
      <td>1.6</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>26.8</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2008-09</td>
      <td>24.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>81.0</td>
      <td>81.0</td>
      <td>3054.0</td>
      <td>9.3</td>
      <td>19.0</td>
      <td>...</td>
      <td>0.780</td>
      <td>1.2</td>
      <td>6.0</td>
      <td>7.2</td>
      <td>6.9</td>
      <td>1.6</td>
      <td>1.1</td>
      <td>2.8</td>
      <td>1.6</td>
      <td>27.2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2009-10</td>
      <td>25.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>76.0</td>
      <td>76.0</td>
      <td>2966.0</td>
      <td>9.3</td>
      <td>18.5</td>
      <td>...</td>
      <td>0.767</td>
      <td>0.9</td>
      <td>5.9</td>
      <td>6.7</td>
      <td>7.9</td>
      <td>1.5</td>
      <td>0.9</td>
      <td>3.2</td>
      <td>1.4</td>
      <td>27.4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2010-11</td>
      <td>26.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>SF</td>
      <td>79.0</td>
      <td>79.0</td>
      <td>3063.0</td>
      <td>8.9</td>
      <td>17.5</td>
      <td>...</td>
      <td>0.759</td>
      <td>0.9</td>
      <td>6.0</td>
      <td>6.9</td>
      <td>6.5</td>
      <td>1.5</td>
      <td>0.6</td>
      <td>3.3</td>
      <td>1.9</td>
      <td>24.8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2011-12</td>
      <td>27.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>SF</td>
      <td>62.0</td>
      <td>62.0</td>
      <td>2326.0</td>
      <td>9.6</td>
      <td>18.1</td>
      <td>...</td>
      <td>0.771</td>
      <td>1.5</td>
      <td>6.2</td>
      <td>7.6</td>
      <td>6.0</td>
      <td>1.8</td>
      <td>0.8</td>
      <td>3.3</td>
      <td>1.5</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2012-13</td>
      <td>28.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>PF</td>
      <td>76.0</td>
      <td>76.0</td>
      <td>2877.0</td>
      <td>9.6</td>
      <td>16.9</td>
      <td>...</td>
      <td>0.753</td>
      <td>1.2</td>
      <td>6.4</td>
      <td>7.6</td>
      <td>6.9</td>
      <td>1.6</td>
      <td>0.8</td>
      <td>2.8</td>
      <td>1.4</td>
      <td>25.5</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2013-14</td>
      <td>29.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>PF</td>
      <td>77.0</td>
      <td>77.0</td>
      <td>2902.0</td>
      <td>9.5</td>
      <td>16.8</td>
      <td>...</td>
      <td>0.750</td>
      <td>1.0</td>
      <td>5.6</td>
      <td>6.6</td>
      <td>6.1</td>
      <td>1.5</td>
      <td>0.3</td>
      <td>3.3</td>
      <td>1.6</td>
      <td>25.9</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2014-15</td>
      <td>30.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>69.0</td>
      <td>69.0</td>
      <td>2493.0</td>
      <td>9.0</td>
      <td>18.5</td>
      <td>...</td>
      <td>0.710</td>
      <td>0.7</td>
      <td>5.3</td>
      <td>6.0</td>
      <td>7.4</td>
      <td>1.6</td>
      <td>0.7</td>
      <td>3.9</td>
      <td>1.9</td>
      <td>25.2</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2015-16</td>
      <td>31.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>76.0</td>
      <td>76.0</td>
      <td>2709.0</td>
      <td>9.8</td>
      <td>18.8</td>
      <td>...</td>
      <td>0.731</td>
      <td>1.5</td>
      <td>6.0</td>
      <td>7.5</td>
      <td>6.8</td>
      <td>1.4</td>
      <td>0.7</td>
      <td>3.3</td>
      <td>1.9</td>
      <td>25.5</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2016-17</td>
      <td>32.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>74.0</td>
      <td>74.0</td>
      <td>2794.0</td>
      <td>9.5</td>
      <td>17.3</td>
      <td>...</td>
      <td>0.674</td>
      <td>1.2</td>
      <td>7.0</td>
      <td>8.2</td>
      <td>8.3</td>
      <td>1.2</td>
      <td>0.6</td>
      <td>3.9</td>
      <td>1.7</td>
      <td>25.2</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2017-18</td>
      <td>33.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>PF</td>
      <td>82.0</td>
      <td>82.0</td>
      <td>3026.0</td>
      <td>10.2</td>
      <td>18.8</td>
      <td>...</td>
      <td>0.731</td>
      <td>1.2</td>
      <td>7.3</td>
      <td>8.4</td>
      <td>8.9</td>
      <td>1.4</td>
      <td>0.8</td>
      <td>4.1</td>
      <td>1.6</td>
      <td>26.8</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018-19</td>
      <td>34.0</td>
      <td>LAL</td>
      <td>NBA</td>
      <td>SF</td>
      <td>55.0</td>
      <td>55.0</td>
      <td>1937.0</td>
      <td>10.4</td>
      <td>20.4</td>
      <td>...</td>
      <td>0.665</td>
      <td>1.1</td>
      <td>7.6</td>
      <td>8.6</td>
      <td>8.4</td>
      <td>1.3</td>
      <td>0.6</td>
      <td>3.7</td>
      <td>1.7</td>
      <td>28.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2019-20</td>
      <td>35.0</td>
      <td>LAL</td>
      <td>NBA</td>
      <td>PG</td>
      <td>39.0</td>
      <td>39.0</td>
      <td>1362.0</td>
      <td>10.0</td>
      <td>20.4</td>
      <td>...</td>
      <td>0.694</td>
      <td>1.1</td>
      <td>6.9</td>
      <td>8.0</td>
      <td>11.2</td>
      <td>1.3</td>
      <td>0.5</td>
      <td>3.9</td>
      <td>1.8</td>
      <td>26.1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Career</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>1237.0</td>
      <td>1236.0</td>
      <td>47597.0</td>
      <td>9.2</td>
      <td>18.3</td>
      <td>...</td>
      <td>0.735</td>
      <td>1.1</td>
      <td>5.8</td>
      <td>6.9</td>
      <td>6.9</td>
      <td>1.5</td>
      <td>0.7</td>
      <td>3.3</td>
      <td>1.7</td>
      <td>25.4</td>
    </tr>
    <tr>
      <th>18</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>11 seasons</td>
      <td>NaN</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>849.0</td>
      <td>848.0</td>
      <td>33130.0</td>
      <td>9.1</td>
      <td>18.5</td>
      <td>...</td>
      <td>0.733</td>
      <td>1.1</td>
      <td>5.6</td>
      <td>6.7</td>
      <td>6.8</td>
      <td>1.5</td>
      <td>0.8</td>
      <td>3.2</td>
      <td>1.8</td>
      <td>25.1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>4 seasons</td>
      <td>NaN</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>294.0</td>
      <td>294.0</td>
      <td>11168.0</td>
      <td>9.4</td>
      <td>17.3</td>
      <td>...</td>
      <td>0.758</td>
      <td>1.1</td>
      <td>6.0</td>
      <td>7.2</td>
      <td>6.4</td>
      <td>1.6</td>
      <td>0.6</td>
      <td>3.2</td>
      <td>1.6</td>
      <td>25.5</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2 seasons</td>
      <td>NaN</td>
      <td>LAL</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>94.0</td>
      <td>94.0</td>
      <td>3299.0</td>
      <td>10.2</td>
      <td>20.4</td>
      <td>...</td>
      <td>0.675</td>
      <td>1.1</td>
      <td>7.3</td>
      <td>8.4</td>
      <td>9.6</td>
      <td>1.3</td>
      <td>0.6</td>
      <td>3.7</td>
      <td>1.8</td>
      <td>27.2</td>
    </tr>
  </tbody>
</table>
<p>22 rows × 29 columns</p>
</div>


 
So that's one approach that works. But why didn't I use this approach?

**Performance**. I found my scraper to be extremely slow, especially when
sending multiple requests. So, I found a workaround. 
 
## Approach 2: Use a static scraper to a very targeted url

If you look at Basketball Reference's tables, that provide a very nice feature
to embed an html table using [Sports Reference Widgets](https://widgets.sports-
reference.com/). Take a peek here:

![Embed table](/assets/2020-01-15-bbref-scraper_2.png)
 
Instead of requesting the page itself, if we send a GET request to the `src` in
the above image, we can get all the content **statically**. Here's an example: 

**In [19]:**

{% highlight python %}
r = get('https://widgets.sports-reference.com/wg.fcgi?css=1&site=bbr&url=%2Fplayers%2Fj%2Fjamesle01.html&div=div_per_minute')
if r.status_code==200:
    soup = BeautifulSoup(r.content, 'html.parser')
    table = soup.find('table')
{% endhighlight %}

**In [20]:**

{% highlight python %}
df = pd.read_html(str(table))[0]
df
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
      <th>Season</th>
      <th>Age</th>
      <th>Tm</th>
      <th>Lg</th>
      <th>Pos</th>
      <th>G</th>
      <th>GS</th>
      <th>MP</th>
      <th>FG</th>
      <th>FGA</th>
      <th>...</th>
      <th>FT%</th>
      <th>ORB</th>
      <th>DRB</th>
      <th>TRB</th>
      <th>AST</th>
      <th>STL</th>
      <th>BLK</th>
      <th>TOV</th>
      <th>PF</th>
      <th>PTS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2003-04</td>
      <td>19.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SG</td>
      <td>79.0</td>
      <td>79.0</td>
      <td>3122.0</td>
      <td>7.2</td>
      <td>17.2</td>
      <td>...</td>
      <td>0.754</td>
      <td>1.1</td>
      <td>3.8</td>
      <td>5.0</td>
      <td>5.4</td>
      <td>1.5</td>
      <td>0.7</td>
      <td>3.1</td>
      <td>1.7</td>
      <td>19.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2004-05</td>
      <td>20.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>80.0</td>
      <td>80.0</td>
      <td>3388.0</td>
      <td>8.4</td>
      <td>17.9</td>
      <td>...</td>
      <td>0.750</td>
      <td>1.2</td>
      <td>5.1</td>
      <td>6.2</td>
      <td>6.1</td>
      <td>1.9</td>
      <td>0.6</td>
      <td>2.8</td>
      <td>1.6</td>
      <td>23.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005-06</td>
      <td>21.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>79.0</td>
      <td>79.0</td>
      <td>3361.0</td>
      <td>9.4</td>
      <td>19.5</td>
      <td>...</td>
      <td>0.738</td>
      <td>0.8</td>
      <td>5.2</td>
      <td>6.0</td>
      <td>5.6</td>
      <td>1.3</td>
      <td>0.7</td>
      <td>2.8</td>
      <td>1.9</td>
      <td>26.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2006-07</td>
      <td>22.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>78.0</td>
      <td>78.0</td>
      <td>3190.0</td>
      <td>8.7</td>
      <td>18.3</td>
      <td>...</td>
      <td>0.698</td>
      <td>0.9</td>
      <td>5.0</td>
      <td>5.9</td>
      <td>5.3</td>
      <td>1.4</td>
      <td>0.6</td>
      <td>2.8</td>
      <td>1.9</td>
      <td>24.1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2007-08</td>
      <td>23.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>75.0</td>
      <td>74.0</td>
      <td>3027.0</td>
      <td>9.4</td>
      <td>19.5</td>
      <td>...</td>
      <td>0.712</td>
      <td>1.6</td>
      <td>5.5</td>
      <td>7.0</td>
      <td>6.4</td>
      <td>1.6</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>26.8</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2008-09</td>
      <td>24.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>81.0</td>
      <td>81.0</td>
      <td>3054.0</td>
      <td>9.3</td>
      <td>19.0</td>
      <td>...</td>
      <td>0.780</td>
      <td>1.2</td>
      <td>6.0</td>
      <td>7.2</td>
      <td>6.9</td>
      <td>1.6</td>
      <td>1.1</td>
      <td>2.8</td>
      <td>1.6</td>
      <td>27.2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2009-10</td>
      <td>25.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>76.0</td>
      <td>76.0</td>
      <td>2966.0</td>
      <td>9.3</td>
      <td>18.5</td>
      <td>...</td>
      <td>0.767</td>
      <td>0.9</td>
      <td>5.9</td>
      <td>6.7</td>
      <td>7.9</td>
      <td>1.5</td>
      <td>0.9</td>
      <td>3.2</td>
      <td>1.4</td>
      <td>27.4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2010-11</td>
      <td>26.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>SF</td>
      <td>79.0</td>
      <td>79.0</td>
      <td>3063.0</td>
      <td>8.9</td>
      <td>17.5</td>
      <td>...</td>
      <td>0.759</td>
      <td>0.9</td>
      <td>6.0</td>
      <td>6.9</td>
      <td>6.5</td>
      <td>1.5</td>
      <td>0.6</td>
      <td>3.3</td>
      <td>1.9</td>
      <td>24.8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2011-12</td>
      <td>27.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>SF</td>
      <td>62.0</td>
      <td>62.0</td>
      <td>2326.0</td>
      <td>9.6</td>
      <td>18.1</td>
      <td>...</td>
      <td>0.771</td>
      <td>1.5</td>
      <td>6.2</td>
      <td>7.6</td>
      <td>6.0</td>
      <td>1.8</td>
      <td>0.8</td>
      <td>3.3</td>
      <td>1.5</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2012-13</td>
      <td>28.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>PF</td>
      <td>76.0</td>
      <td>76.0</td>
      <td>2877.0</td>
      <td>9.6</td>
      <td>16.9</td>
      <td>...</td>
      <td>0.753</td>
      <td>1.2</td>
      <td>6.4</td>
      <td>7.6</td>
      <td>6.9</td>
      <td>1.6</td>
      <td>0.8</td>
      <td>2.8</td>
      <td>1.4</td>
      <td>25.5</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2013-14</td>
      <td>29.0</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>PF</td>
      <td>77.0</td>
      <td>77.0</td>
      <td>2902.0</td>
      <td>9.5</td>
      <td>16.8</td>
      <td>...</td>
      <td>0.750</td>
      <td>1.0</td>
      <td>5.6</td>
      <td>6.6</td>
      <td>6.1</td>
      <td>1.5</td>
      <td>0.3</td>
      <td>3.3</td>
      <td>1.6</td>
      <td>25.9</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2014-15</td>
      <td>30.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>69.0</td>
      <td>69.0</td>
      <td>2493.0</td>
      <td>9.0</td>
      <td>18.5</td>
      <td>...</td>
      <td>0.710</td>
      <td>0.7</td>
      <td>5.3</td>
      <td>6.0</td>
      <td>7.4</td>
      <td>1.6</td>
      <td>0.7</td>
      <td>3.9</td>
      <td>1.9</td>
      <td>25.2</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2015-16</td>
      <td>31.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>76.0</td>
      <td>76.0</td>
      <td>2709.0</td>
      <td>9.8</td>
      <td>18.8</td>
      <td>...</td>
      <td>0.731</td>
      <td>1.5</td>
      <td>6.0</td>
      <td>7.5</td>
      <td>6.8</td>
      <td>1.4</td>
      <td>0.7</td>
      <td>3.3</td>
      <td>1.9</td>
      <td>25.5</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2016-17</td>
      <td>32.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>SF</td>
      <td>74.0</td>
      <td>74.0</td>
      <td>2794.0</td>
      <td>9.5</td>
      <td>17.3</td>
      <td>...</td>
      <td>0.674</td>
      <td>1.2</td>
      <td>7.0</td>
      <td>8.2</td>
      <td>8.3</td>
      <td>1.2</td>
      <td>0.6</td>
      <td>3.9</td>
      <td>1.7</td>
      <td>25.2</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2017-18</td>
      <td>33.0</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>PF</td>
      <td>82.0</td>
      <td>82.0</td>
      <td>3026.0</td>
      <td>10.2</td>
      <td>18.8</td>
      <td>...</td>
      <td>0.731</td>
      <td>1.2</td>
      <td>7.3</td>
      <td>8.4</td>
      <td>8.9</td>
      <td>1.4</td>
      <td>0.8</td>
      <td>4.1</td>
      <td>1.6</td>
      <td>26.8</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018-19</td>
      <td>34.0</td>
      <td>LAL</td>
      <td>NBA</td>
      <td>SF</td>
      <td>55.0</td>
      <td>55.0</td>
      <td>1937.0</td>
      <td>10.4</td>
      <td>20.4</td>
      <td>...</td>
      <td>0.665</td>
      <td>1.1</td>
      <td>7.6</td>
      <td>8.6</td>
      <td>8.4</td>
      <td>1.3</td>
      <td>0.6</td>
      <td>3.7</td>
      <td>1.7</td>
      <td>28.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2019-20</td>
      <td>35.0</td>
      <td>LAL</td>
      <td>NBA</td>
      <td>PG</td>
      <td>39.0</td>
      <td>39.0</td>
      <td>1362.0</td>
      <td>10.0</td>
      <td>20.4</td>
      <td>...</td>
      <td>0.694</td>
      <td>1.1</td>
      <td>6.9</td>
      <td>8.0</td>
      <td>11.2</td>
      <td>1.3</td>
      <td>0.5</td>
      <td>3.9</td>
      <td>1.8</td>
      <td>26.1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Career</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>1237.0</td>
      <td>1236.0</td>
      <td>47597.0</td>
      <td>9.2</td>
      <td>18.3</td>
      <td>...</td>
      <td>0.735</td>
      <td>1.1</td>
      <td>5.8</td>
      <td>6.9</td>
      <td>6.9</td>
      <td>1.5</td>
      <td>0.7</td>
      <td>3.3</td>
      <td>1.7</td>
      <td>25.4</td>
    </tr>
    <tr>
      <th>18</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>11 seasons</td>
      <td>NaN</td>
      <td>CLE</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>849.0</td>
      <td>848.0</td>
      <td>33130.0</td>
      <td>9.1</td>
      <td>18.5</td>
      <td>...</td>
      <td>0.733</td>
      <td>1.1</td>
      <td>5.6</td>
      <td>6.7</td>
      <td>6.8</td>
      <td>1.5</td>
      <td>0.8</td>
      <td>3.2</td>
      <td>1.8</td>
      <td>25.1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>4 seasons</td>
      <td>NaN</td>
      <td>MIA</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>294.0</td>
      <td>294.0</td>
      <td>11168.0</td>
      <td>9.4</td>
      <td>17.3</td>
      <td>...</td>
      <td>0.758</td>
      <td>1.1</td>
      <td>6.0</td>
      <td>7.2</td>
      <td>6.4</td>
      <td>1.6</td>
      <td>0.6</td>
      <td>3.2</td>
      <td>1.6</td>
      <td>25.5</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2 seasons</td>
      <td>NaN</td>
      <td>LAL</td>
      <td>NBA</td>
      <td>NaN</td>
      <td>94.0</td>
      <td>94.0</td>
      <td>3299.0</td>
      <td>10.2</td>
      <td>20.4</td>
      <td>...</td>
      <td>0.675</td>
      <td>1.1</td>
      <td>7.3</td>
      <td>8.4</td>
      <td>9.6</td>
      <td>1.3</td>
      <td>0.6</td>
      <td>3.7</td>
      <td>1.8</td>
      <td>27.2</td>
    </tr>
  </tbody>
</table>
<p>22 rows × 29 columns</p>
</div>


 
Just as good! Note that this workaround only works for Sports Reference websites
and will not work for other website with dynamically loaded content. In that
case you will have to use another dynamic scraper a la Selenium or Pyppeter. 
 
## Results

My Basketball Reference Scraper is now fully functional and highly used across
many NBA data enthusiasts. Here is the final product:

[basketball\_reference\_scraper](https://github.com/vishaalagartha/basketball_re
ference_scraper)

**An API client to access statistics and data from** [**Basketball
Reference**](https://www.basketball-reference.com/) **via scraping written in
Python.**


    pip install basketball-reference-scraper==v1.0.1

All the methods are documented [here](https://github.com/vishaalagartha/basketba
ll_reference_scraper/blob/master/API.md) along with [examples](https://github.co
m/vishaalagartha/basketball_reference_scraper/blob/master/examples.py).

Please feel free to check out the [GitHub
repo](https://github.com/vishaalagartha/basketball_reference_scraper) as well.

Anyone is more than welcome to create issues regarding any problems that you may
experience. I will try my best to be as responsive as possible. Please feel free
to provide criticism as I would love to improve this even further! 

**In [None]:**

{% highlight python %}

{% endhighlight %}
