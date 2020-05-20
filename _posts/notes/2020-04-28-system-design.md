---
title: "CS Interview Prep Part 5: System Design"
date: 2020-04-28
permalink: /notes/2020/04/28/system-design
tags:
--- 

# Design a service like TinyURL

- Url's can have characters `'A-Za-z0-9'`, this creates 62 possible characters. Assuming we need 1,000 billion URLs, we can have the `short_url` have a suffix of 7 characters
- We need a way of uniquely hashing the `short_url`'s in a database. To achieve this, have an auto incrementing ID in base 10 that we convert to base 62. Now every url is unique automatically. 

Data Model
```
Table Tiny_Url(
ID : int PRIMARY_KEY AUTO_INC,
long_url: varchar,
short_url: varchar
)
```

# Design Video Streaming like Netflix

* Control Plane
  - Content uploader uploads to storage service
  - Storage service segments video and puts onto storage server (e.g. Amazon S3)
  - Storage server transcodes into different formats for all devices/platforms and puts back into storage server
  - Storage server pushes to CDNs to optimize user experience while delivering content
* Data Plane 
  - Content play request hits playback service, forwarded to authentication service, forwarded to steering service to choose CDN
  - User gets video from CDN
  - User publishes events from playback experience to playback experience service


# Design Messaging Service like WhatsApp

* One to One messaging
  - A sends message to B, which is pushed to chat service, upon which A receives delivered message
  - Chat data storage locates B's server and checks if B online
  - If B online, message sent to B
  - Otherwise, B stores message in transient data storage
  - Once B receives, B sends received message back to chat service which works in same way

* Check whether user is online
  - Store Last Seen timestamp whenever user sends request
  - If Last Seen timestamp is within 20 seconds, indicate user is 'online'

* Group Messaging
  - Chat service will look into group service to see which users are in another group
  - Chat service forwards all messages to users

Other notes:
  - Decrease load on gateways using Parser/Unparser like Thrift
  - Use Consistent Hashing to store Group ID to User ID mapping

# Design a Newsfeed like Twitter/Instagram/Facebook

* Newsfeed
  - User sends HTTP request to gateway, which uses a load balancer to asks for user feed service
  - User feed service asks follow service for followers, then looks to posts service for top posts

# Design Ride Sharing Service like Uber

* Trip Storage
  - Non relational database
  - Backup with multiple datacenters and caches to improve latency
  - Keep archive for analytics

* Customer Driver Matching and Mapping Service
  - Computing ETA, route requires solving the Traveling Salesman Problem (NP-Complete). 
  - Use historical data for ETA calculation and split city into smaller parts and use graph algorithm (A* or Dijkstra)
  - Use car metrics, ratings, etc. to incorporate Driver Matching 

# Design Typeahead/Autocomplete Feature

* Use a trie datastructure (optimizes space and time)
  - Give each leaf a ranking
  - Follow a query along the trie and find all children
  - Sort all children by ranking

* Caching using prefix hashed table
  - NoSQL since relationships are low

# Design File Sharing System like Dropbox/Google Drive

* Upload/Download
   - Break files into smaller chunks to reduce bandwidth usage, cloud usage, and parallize processes
   - Store information of different versions using metadata file
   - Store on CDNs to reduce latency of download

# Design API Rate Limiter

* Algorithms
  - Leaky Bucket - FIFO queue where elements added when queue is at max capacity are dropped (i.e. leaked)
  - Fixed Window - limit the number of requests a user can make for a fixed window size (e.g. 2 req/min)
  - Sliding Log - remove outdated requests based on fixed size 
