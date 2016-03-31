---
layout: post
title:  "Things Slow Down Your Server"
date:   2016-01-27 14:09:21
author: Yang
---
When I worked in the backend team in BloomSky, we built our backend using Django Rest framework, and deployed the backend in Google App Engine. We used services provided by Google such as Cloud SQL and Compute Engine. Everything was fine at first until we reach a point when we had 30000 users and 2000 smart weather stations with our backend. All the APIs began to suffer from periodic high latency, in some severe situation, the requests to our server began to time out.

To find out the problem with our backend, we enabled the slow log in the 
Cloud SQL, you can check out [Configuring MySQL Flags][Configuring-MySQL-Flags] to learn how to enable slow log on Cloud SQL. With slow log enabled, we found
that some sql queries took 30s to finish. Why it took such a long time for 
a query to run, it turned out that we had a many to many table that recorded
the following relationship, and we calculated the number of followers using this table. To solove the problem, we cached the number of followers which significatly reduced the burden of the database and made the latency of our APIs back to normal.

The lession learned is never do expensive calculation in database, and you can always enable slow log in SQL to find out what slow down your server.



[Configuring-MySQL-Flags]: https://cloud.google.com/sql/docs/mysql-flags
