---
title: "Threadalytics - An NBA Sentiment Analysis Platform"
date: 2024-01-17
permalink: /projects/2024/01/17/threadalytics
tags:
    - Projects
--- 

The goal of this project was to capture the recent sentiments from r/nba about NBA players and teams.
After capturing relevant data, we want to display it in a visually appealing fashion and convey details about why a certain team or player got the score.

The final product is available at [Threadalytics](https://threadalytics.com).

Based on this we can capture the following functional and non-functional requirements

## Functional Requirements
* Objectively calculate sentiment of all NBA teams and players over last day
  * Display overall subreddit sentiment
* Show ratio of positive to negative titles over course of day
* Show compound sentiment 'score' as a function of time
* Display the most mentioned players and teams
  * With ability to toggle date
* Display top player sentiment scores and update every hour
  * With ability to toggle date
  * Show some sentences that contributed to the score
  
Non-Functional Requirements
* Performance/Latency - Users should not have to wait a long time to see results
* Usability - Graphical representation should be aesthetic and intuitive to understand

Based on these requirements, we can justify the following tech stack:
* ReactJS (frontend framework)
* D3JS (data visualization)
* NodeJS (backend framework)
* Python (scraping and sentiment analysis)
* AWS S3 (static website hosting)
* AWS EC2 (server hosting)
* AWS S3 (log/database)
* Route53 + Cloudfront (CDN, domain hosting, and SSL certificate management)
Let's dive a little deeper into some specifics:



We can opt to use a microservice architecture in this situation to reduce the load across multiple servers. Namely, we will have multiple services performing:
* Scraping titles and comments and performing sentiment analysis on player, team, and coach names
* Scraping titles to understand the overall sentiment of the subreddit
* Finding the top comments of each day
* Logging the most frequently mentioned team and player of the day
  
Of course, we will have a main API server that will handle all client HTTP requests.

## AWS Infrastructure
I opted to use AWS Lambda to handle the scraping and sentiment analysis logic. AWS Lambda is a serverless solution that allows users to call lambda 'functions' on a given trigger. These functions can be written in multiple languages such as NodeJS, Python, Java, etc. and can be further customized using containerization and lambda layers.
Since my application used PRAW, a Python API wrapper and NLTK's Python library, it was simple to use a Python3.8 runtime environment. However, I did need multiple packages to be installed prior to function execution, so I created a custom lambda layer. The lambda layer consisted of the following packages necessary to perform scraping, sentiment analysis, and writing to S3: PRAW - for scraping, NLTK - for sentiment analysis, boto3 - for writing to S3 and other AWS interaction

We can use AWS's Event Scheduler service to run each of these as a CRON job. The Player and Overall Sentiment services will run on an hourly basis, while the Top Comments and Daily Winners services will run on a daily basis.

### Player sentiment analysis microservice
This module is responsible for obtaining sentiments toward all players over the past hour. The logic is as follows:

1) Obtain top posts for 400 seconds.
For each post, obtain all comments with a limit of 1000, sorted by most upvotes

2) Use a prewritten list of all NBA players, teams, and coaches to find sentences containing any entity.
For each entity maintain a list of sentences that involve them using a hashmap

3) Sort the hashmap to find the most frequently mentioned entities
4) 
5) For each entity in the top 10, compute a polarity score of each of its sentences.
If the polarity score is excessively positive or negative, store it for future use. If the number of positive scores is much larger than the number of negative scores (or vice a versa) store the entity's scores and sentences

6) Write results to S3 log file

This service runs on an hourly basis


### Subreddit Sentiment Microservice
This module is responsible for obtaining overall sentiment of the subreddit The logic is as follows:

1) Obtain all posts over last hour. For each post, compute the polarity score of the title and aggregate it

2) Write results to a S3 log file

This service runs on an hourly basis

### Top Comments Microservice
This module is responsible for obtaining overall sentiment of the subreddit The logic is as follows:

1) Obtain the top posts of the day.
For each post, get the most upvoted comment

2) Sort comments by upvotes

3) Write results to a S3 log file

This service runs on an hourly basis

### Daily Winners Microservice
This module is responsible for obtaining overall sentiment of the subreddit The logic is as follows:

1) Obtain the top posts of the day.
For each post, get the top 1000 comments

2) Count the number of occurrences of each player and team

3) Sort by most occurrences

4) Obtain most mentioned player and most mentioned team

5) Write results to a log file

This service runs on an hourly basis

### Deployment architecture
S3 buckets will maintain our static website, which can be synced and easily deployed. The domain name threadalytics was purchased via NameCheap, but we can create a hosted zone which will allow us to link our domain name to our S3 bucket by pointing the domain to the provided name servers.
Additionally, AWS certificate manager will allow us to create certificates for all *.threadalytics.com endpoints. This will allow us to host our frontend on https://threadalytics.com and https://www.threadalytics.com.
The subdomain https://api.threadalytics.com will host our API.


### API Design
Endpoint: https://api.threadalytics.com

We will need 4 endpoints for our server:
* GET /sentiments
* GET /scores
* GET /comments
* GET /winners

Each endpoint will read from the appropriate log file stored on S3 and return the appropriate results

[Please refer to this FigJam file for the full deployment architecture](https://www.figma.com/file/Z1puaOabg8GjFQlxAKSos0/Threadalytics-System-Design?type=whiteboard&node-id=0%3A1&t=WKmPR9yLw7fZu7sh-1)