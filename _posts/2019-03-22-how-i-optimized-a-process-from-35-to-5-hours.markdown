---
layout: post
title:  "How I optimized a process from 35 to 5 hours"
date:   2019-03-22 17:00:00 +0100
permalink: /2019/03/22/how-i-optimized-a-process-from-35-to-5-hours/
tags:
  - optimization
  - sql
---

Most of my day job isn’t fascinating. Yet another controller, service, test, and so on. I spent a lof of time doing repetitive tasks and slowly gaining more knowledge about the system I’m working on. However, having slow and steady pace can eventually reward you with an opportunity to make a really great improvement. It happened to me, twice.

I spent four years maintaining a project with a 65 GB MySQL database and hundreds of millions of records. In the beginning, the system seemed to be very complex. It contained lots of legacy code and many classes turned out to be obsolete. I needed some time to raise my confidence with this project. After two years, I reduced the database size to 15 gigabytes without any data loss. Of course during my work, the database gained another millions of records, but that didn’t stop me from doing a stunning optimization.

It wasn’t a single database migration, but a series of small ones. It took me many months to come up with all the optimizations, sometimes subtle – but with 300 millions records, every byte counted. Database schema changes required also application code changes and I did not want to make too big PRs. Moreover, I couldn’t just execute a migration on a 60 GB table whenever I wanted. I had to agree with the Product Owner on a downtime. And, of course, I had to prepare backups and a rollback strategy.

Then I jumped to work on an advertising platform which had a complex invoicing system. Every night, a cron job was run to create and send PDF invoices. The process was supposed to finish in the morning, but it didn’t. People did not get their documents in time. I discovered that a single process can run up to 35 hours, even if just a few documents were made.

Again, I had to do several boring maintenance tasks before I had the courage to optimize the complex invoicing process. After gaining some basic system knowledge I noticed that the cron job did not have any tests. Every change required manual tests. So I spent some time writing unit and integration tests which helped me understand the process even more.

When I was ready to introduce changes, I talked to the Product Owner and he agreed to include that work in the upcoming sprint. I needed two weeks to do necessary measurements and experiments. In the end, I successfully deployed my changes and the process shrinked from 35 to 7 hours. I removed a lot of redundant database queries by simply verifying the boolean logic and the control flow. Are you aware of the way boolean expressions are evaluated? Knowing such nuts and bolts might really reward you.

## How to conduct a successful process optimization

1. **Know the details** of the business logic and code details of the project you are working on. To do so, maintain a steady workflow. Don’t be afraid of boring tasks – you have to start somewhere.
2. **Test the current behavior** that you plan to optimize. Write automatic tests or at least prepare manual scenarios to cover as much cases as possible (not just happy paths). This will expand your project knowledge even further.
3. **Measure all steps** of the current process. Which step is the slowest and why? How much time on average it takes to process a single entity? You need measures to know if you’re making any progress.
4. **Introduce changes** in the code and verify them on your local or test environment.
5. **Gather feedback** during code review. Maybe someone will notice dangerous changes you overlooked.
6. **Analyze** what is needed for deployment. Will it require downtime? Any database migrations? Are there any other applications depending on your module/service/app?
7. **Practice** deployment in a test or pre-production environment.
8. **Discuss** the deployment time with the team and stakeholders.
9. Good luck! Go with the deployment :)
