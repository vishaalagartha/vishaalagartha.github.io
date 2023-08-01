---
title: "CS Interview Prep Part 6: System Design Primer"
date: 2023-07-12
permalink: /notes/2023/07/12/system-design-primer
tags:
--- 
# System Design Primer 
 
## Scalability Basics 
 
### Vertical Scaling - upgrade machine to have more RAM, cores, disks, etc. 
 
### Horizontal Scaling - getting more machines

In order to scale horizontally, we want to make sure we have the same codebase
on all our servers. So how can we change code on all servers at once?

Using **clones**. Once we make one server, we can create an image (in AWS terms
an AMI) which all our servers will launch from whenever they get spun up. 


#### Map Reduce

If we have multiple machines, but a massive job, how do we split it up? Via MapReduce.

1) Map - Split input into pieces and perform job

2) Reduce - Combine results of map phase into one result
 
### Databases

**Replication** means replicating the database contents to improve availability.
This concept is similar to **denormalization**, a process to speed up SQL reads
at the cost of SQL writes.

To improve scalability, we can switch to a NoSQL database and just perform more
complex queries in application code.

**Partitioning** - splitting the database into multiple pieces

* Caching - a way of storing information for faster retrieval

Method 1: Cached queries - hash a DB query and save it in the cache, but makes
it hard to delete items

Method 2: Cached objects - store entire objects in cache, making asynchronous
processing possible 
 
### Load Balancing

Away of distributing queries to servers to balance the load 
 
### Reverse Proxy

Similar to a load balancer, a reverse proxy is a middleman between the client
and server. They are also used in the case of a single server to improve
performance, security, and flexibility. 
 
### Asynchronism

Method 1: Precomputing and turning dynamic content into static content to be
delivered quickly. But, this doesn't work if user wants something customized.

Method 2: Use a queue to maintain a list running and completed jobs. 
 
## CAP Theorem

A distributed can support only 2 of the following:

1) Consistency - every read receives the most recent write
2) Availability - every request receives a response (that may not be the most
recent)
3) Partition tolerance - system works even if network fails

Partition tolerance is a must since networks are unreliable, so we have to give
up C or A.

CP - if we want atomic reads/writes guaranteed

AP - system MUST keep working 
 
## Domain Name System (DNS)

How to translate a domain name (www.example.com) to an IP address?

1) User asks for domain name

2) Local DNS server asks root DNS server for IP of domain name

3) Root DNS server sends back response

4) Local DNS servers will save response in cache and forward to user

5) User will get IP, cache the IP, and ask IP directly 
 
## Content Delivery Network (CDN)

Servers located across the world that serve (usually) static content. Benefits
include:

1) reduce the amount of requests your server needs to handle

2) allow users to obtain content more quickly due to proximity 
 
## Databases (in-depth)

#### Relational Database Management System (RDBMS)

SQL Databases that are structured like tables. They follow the ACID (atomicity,
consistency, isolation, and durability) principle.

How to scale an RDBMS?
- Master-Slave Replication
Master serves reads and writes and copies data to slaves. Slaves only serve
reads.

Now, if a master goes down, we're in read-only mode. We also need a load
balancer to handle reads.

- Master-Master Replication
Multiple masters to serve both reads and writes. They communicate to one
another.

Requires a lot of extra logic on how to serve requests and communication.

- Federation
Separate database by service functionality (e.g. forums, users, products). This
reduces the load on each database server.

- Sharding
Similar to federation, except don't break servers apart by functionality.

- Denormalization
Make multiple copies of the data to reduce time to reduce the number of joins
when serving requests. Improves read performance at the cost of write
performance.

- SQL Tuning
Improving database schema.

#### NoSQL

Data is stored as keys and values (like a Python dictionary or JSON). Data is
denormalized and joins are now performed in the application layer.

### SQL vs. NoSQL

SQL pros:
- structured data
- more established
- want data to be consistent
- easy to scale

NoSQL pros:
- flexible data
- fast queries

### Consistent Hashing

How to add servers to your architecture without losing valuable cached data?

For example, we have 3 servers and user X is being served by server 1:

```
hash(X) % 3 = 1
```

Now, if we add another server, the modulo operator messes things up!

```
hash(X) % 4 !=1
```

And we lose the valuable cached information.

Instead, imagine our servers in a clockwise ring and requests are being served by the next server clockwise to it.

Now, if we add a server, the only server being affected is the one ONLY after.
 
 
## Communication

### HTTP

Method of transmiting data between a client and a server. Operates via TCP (less
time critical, but reliable) or UDP (lower latency, not guaranteed)

### RPC

Method of asking a server to perform a procedure for the client (like AWS
lambda). 

**In [None]:**

{% highlight python %}

{% endhighlight %}
