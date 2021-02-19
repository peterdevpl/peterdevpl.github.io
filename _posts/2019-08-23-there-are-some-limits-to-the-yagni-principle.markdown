---
layout: post
title:  "There are some limits to the YAGNI principle"
date:   2019-08-23 17:00:00 +0100
description: The biggest challenge for me has been always dealing with complex business logic in IT systems. You need good tools to model and test that logic.
permalink: /2019/08/23/there-are-some-limits-to-the-yagni-principle/
tags:
  - management
  - php
  - team leader
---

Have you heard of YAGNI? The acronym stands for [“You Ain’t Gonna Need It“](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it). The original meaning, as Extreme Programming guru Ron Jeffries said, was to “implement things when you actually need them, never when you just foresee that you need them”. But how did this principle evolve over time?

Since the late 90’s we have witnessed a lot of frameworks and tools emerge. We love new shiny tools, don’t we? Sometimes we think that a new framework will solve all our problems. That’s what the [“Hype Driven Development”](https://blog.daftcode.pl/hype-driven-development-3469fc2e9b22) article described.

So then people started to talk: okay, maybe we should care more about the business, not just our fun? Maybe we don’t need so many tools and libraries everyone else is hyped about? Maybe we should not do a big rewrite of our systems every six months after another tech conference?

## Don’t do everything by yourself

If using the latest hyped tool for every task is one extreme, then doing everything by yourself is another one. The latter option was proposed by some guy conducting a training on microservices (another hype term by the way) for my company.

I was in a team using PHP and we wanted to know something more about developing proper microservices architecture. We did a lot of analysis of our current system and decided this might be a chance. What we heard during the training was:

* Our coach would never use Symfony for a microservice because “it’s huge and will cause performance issues”. That was a couple of months before Symfony 4 was released, but the same guy argued that “computing power is cheap and we shouldn’t care”. Umm…
* Our coach would never use Doctrine for a microservice because “it’s huge and will cause performance issues”.
Instead of showing us some alternatives, the coach asked us to write an example application in vanilla PHP.

By the time the training was conducted, I had already spent years working on home-brew “frameworks”, made by people who did not believe in the popular systems and desperately wanted to do things their own way. Such people tend to leave the company a couple of years later, overwhelmed by all the issues caused by their “ingenious” frameworks.

## Pick tools carefully, get to know them well

So you’re afraid of incorporating third-party code into your project? Cool, it means you’re a responsible developer. But before you turn down popular and well-tested solutions, get to know them well. How much can you fine-tune them? How much can they be customized and stripped of unnecessary features?

Back to PHP stuff, [Symfony](http://symfony.com/doc/current/performance.html) is highly customizable especially since version 4. Symfony and Doctrine tend to cache a lot of things. You can also optimize Composer’s autoloader.

## Don’t optimize prematurely

Performance can be easily improved. But business logic can be a real monster.

After a couple of years working on big, sophisticated projects I realized that performance is a secondary issue. The biggest challenge for me has been always dealing with complex business logic, huge number of moving parts dependent on each other, unclear rules etc. You need good tools to model and test that logic.

So what you shortened page loading time from 300 to 100 milliseconds if your teammates do not understand the crazy optimizations you just made? What benefit do you have after forcing your team to use weird tools they will complain about?

You need to balance these things out.
