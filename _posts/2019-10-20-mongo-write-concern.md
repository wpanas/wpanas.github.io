---
title: How to setup MongoDB write concern?
categories: ['programming']
---

[MongoDB](https://www.mongodb.com/) is flexible NoSQL database. It stores data as documents - data structures based on name-value pairs. Values can be nested name-value pairs, arrays of values or simple scalars. This allows you to build complex data structures.

MongoDB allows you to setup when you consider data to be written.
It might seem strange to you, but let me explain, why this option is even available.

First of all, when you save data to Mongo, it doesn't have to be save on-disk. 

// todo describe journaling

// todo describe acknowlagement

# Standalone

On-disk journal is the only option available in single `mongod` environment. There is no other nodes, that can acknowledge a write operation.

# Cluster

If you consider using MongoDB or any NoSQL database, you should know about [Brewer's theorem](https://en.wikipedia.org/wiki/CAP_theorem). 
Eric Brewer states that network partitioned database can't be simultaneously available and consistent.

It means that if your distributed database is consistent, it will not be available all the time. It might happen, that responding to your query, will took longer. Database will proof that data -- that is answered to your query -- is the freshest.

If your database is focused on availability, it will answer you faster. Yet, you won't be sure that data that you received is the most recent.

MongoDB is focused on availability.

# Shards

# Simultaneous availability and consistency

As I mentioned before, Eric Brewer states that network partitioned database can't be simultaneously available and consistent. 
If you need availability and consistency, then you should consider dropping network partitioning.

Network partitioning is not the only way, to achieve scalability. MongoDB gives you ability to partition your database in shards. Each shard has it's own master node - a primary - and replicas nodes - secondaries.

# Multi-document transactions