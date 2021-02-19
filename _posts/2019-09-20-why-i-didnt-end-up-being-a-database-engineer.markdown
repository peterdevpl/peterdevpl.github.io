---
layout: post
title:  "Why I didn’t end up being a database engineer"
date:   2019-09-20 17:00:00 +0100
description: A story about why you shouldn't put all the business logic in a SQL database.
permalink: /2019/09/20/why-i-didnt-end-up-being-a-database-engineer/
tags:
  - domain-driven design
  - sql
---

Once upon a time, looking for new career opportunities, I considered becoming a database engineer or architect. I was pretty successful in optimizing MySQL databases growing up to 65 GB. I liked it. However I noticed almost no one was hiring MySQL database engineers – it’s too simple. Real challenges await those familiar with Oracle and big finance systems, aviation etc. So, I bought a [thick book about Oracle](https://www.amazon.com/Oracle-Database-12c-Jason-Price-ebook/dp/B00C4BDR62) and… never used it.

The only profit I got from buying an Oracle guide was another book I was offered in package: [“SQL Antipatterns: Avoiding the Pitfalls of Database Programming”](https://www.amazon.com/SQL-Antipatterns-Programming-Pragmatic-Programmers/dp/1934356557) by Bill Karwin. It provided me a lot of valuable tips which I still eagerly keep on sharing with my coworkers.

## Is database the center of every app?

Anyway, I used to believe that a [database is in the center of every application](https://www.youtube.com/watch?v=o_TH-Y78tt4&feature=youtu.be&t=2570). Most of them process data, right? So I started all projects from conceiving a database model: relations, data types, queries, and so on. I spent a lot of time optimizing data structures.

My approach had several pros. First of all, it was intuitive for other developers because a good database model made them immediately understand the purpose of the system and navigate easily through the rising codebase. Secondly, a good model helped datasets grow significantly (like hundreds of millions of records) without performance loss. Moreover, with correct foreign keys and other constraints, data remained consistent.

I have a funny story about a team of developers which refrained from using foreign key constraints. They used `ON DELETE CASCADE` clause everywhere and they accidentally wiped away half of the test database. It was such a shock for them they removed all the foreign keys. Soon, data became terribly inconsistent and a lot of bugs surfaced in the apps depending on that database.

## Carving a SQL giant

Optimizing databases was cool, but there was one bad practice I really disliked looking at some DBAs work. After I read about SOLID principles, Domain-Driven Design and other stuff, I started to hate SQL stored procedures, especially when they were too long.

I saw some MS SQL devs trying to handle all the business logic with SQL scripts. These systems already had many bugs, and my fellow devs just kept on adding more `CASE…IF` blocks after requests from the business people. After some time, it was nearly impossible to add or change anything without breaking stuff.

Such situations made me realize that you have to choose the right tools for the job, not the other way around. Of course data is a huge asset for every company, but behavior is important too. Is SQL a right tool to model complex business behaviors?

## Getting to know the big picture

For me, data has never been the biggest challenge. It is always the sophisticated business logic that needs to be modelled very carefully. It evolves over time, and we have to make sure all developers (especially the new ones) dig it quickly and flawlessly.

I decided to improve my process modelling skills with OOP and FP, know SOLID, DDD and other widely adopted principles or design patterns. I decided to share business knowledge with all my teammates to make sure they understand the purpose of their work and nature of the surrounding business. Of course SQL knowledge is valuable too, but it’s not enough.
