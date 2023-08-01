---
title: "Software Architecture Fundamentals"
date: 2023-03-31
permalink: /notes/2023/03/31/software-architecture
tags:
--- 

# Architecture Patterns

- Client-server
  - Client sends request, server sends response
  - Ex: Any website
- Peer-to-peer
  - Peers/nodes communicate to one another without a central server
  - Ex: blockchain technology, BitTorrent
- Model-View-Controller (MVC)
  - Models describe how data is stored
  - Views create display for user
  - Controller interfaces between model and view
- Microservice
  - Different features split into separate modules that communicate with one another
  - Easier/cleaner/faster maintenance than monolithic architecture
- Event driven
  - Non blocking architecture for handling concurrent connections
  - Ex: modern web development
- Layered
  - Architecture with multiple layers of abstraction
- Hexagonal
  - Consists of Ports, Adapters, Domains
  - Loosely coupled and easy to test

# Number of Tiers
- Single tier for low network latency and no requests
- Two tiers to minimize network latency with minimal requests
- Three tiers/N-tier to have separate and full control over data

# Horizontal vs. Vertical Scaling
- Vertical scaling: when you decide to take one server and make system more powerful
  - Use if there isn't too much traffic and don't need to balance requests
- Horizontal scaling: when you decide to add **more** servers to make system more powerful
  - Use if availability and # of requests expected to increase

# Monolithic vs. Microservice

Monolithic architecture for simple applications that don't need to scale very much. There aren't going to
be very many new features or complexity added.

Microservice architecture for complicated applications with multiple components that need to scale
quickly and fast.

# SQL vs. NoSQL

SQL (relational) databases are tables
  - Use for data consistency, storing relationships, and speed
  - Use for data that needs to **fast**, but relatively the same across (it's hard to add a column to a SQL db) 
  - Examples: MySQL, PostgresSQL, (kind of like a Pandas dataframe)

NoSQL (nonrelational) databases are key-value stored
  - Use for lots of requests and scalability
  - Use for unstructured data that is intuitive (it's easy to add key-value pairs) 
  - Examples: MongoDB, Redis, JSON
