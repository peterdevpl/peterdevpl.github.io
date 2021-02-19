---
layout: post
title:  "Do we still need recruitment agencies?"
date:   2018-11-16 17:00:00 +0100
description: A rant on recruitment agencies doing a bad job. Things to watch for when you are outsourcing your recruitment process.
permalink: /2018/11/16/do-we-still-need-recruitment-agencies/
tags:
  - hr
  - management
  - php
  - recruitment
---

*Article originally published on [dev.to](https://dev.to/phorzycki/do-we-still-need-recruitment-agencies-4gpk)*

As a candidate looking for jobs, I’ve never cooperated with any recruitment agencies. But as a senior developer responsible for tech interviews, I was forced to work with some HR companies and I had some really weird situations because of that. Sometimes it’s annoying, sometimes it just makes me laugh. Anyway, due to my bad experiences it’s hard for me to find any reason to pay commission to an HR/talent agency. I’ve always disliked “men-in-the-middle”. Yet, many companies seek talents through agencies in addition to their own headhunting struggles.

Let me briefly explain a standard recruitment process we developed in our company. We didn’t have an HR department, so the whole process was led by me and CTO. First, I prepared a job offer which would reflect our current needs. Then, the CTO posted that offer on various platforms. We received different resumes and reviewed them. If you did not have a meaningful GitHub profile, we usually asked to do our simple recruitment task: write a PHP shopping cart implementation for existing unit tests; kind of TDD, most people solved it under 4 hours. We also sent some technical questions to see if we don’t waste your (and our) time meeting you at the office. Especially if you lived several hundred miles away. If we liked your answers and the solution to our task, we would invite you to a personal meeting (around 1.5 hour) consisting of a soft interview and tech interview. Finally, you could receive an offer from us… and turn it down if you did not like it.

Simple, isn’t it? Just two parties making business with each other. We are the client and we buy services that you provide. We negotiate the deal and if everyone’s happy, just tell us when you can start. The description I mentioned is generic; we tried to approach every candidate individually according to the provided materials, proven experience and a lot of other factors. You didn’t have to solve our coding task if you found another way to show off. Fun fact: we’ve never cared about your formal education.

Unfortunately the resumes we received from job board users were not enough and because the company did not want to invest its money in greater headhunting endeavors like going to conferences or recruiting a full-time HR guy, well… it decided to use some help from external HR agencies. We would present our needs to the agency and it was supposed to find right people for the job. We received a written recommendation for every candidate. After successfully hiring an employee, the company paid commission to the agency.

## How recruiting with agencies went wrong

I wouldn’t moan if those companies did their job well, but what I experienced instead was:

1. The recommendations were generic and useless. Every candidate was described as a positive, pro-active team player who aims for personal development. Blah, blah, blah. If you stripped that BS, you would find out that you’re dealing with a mediocre coder with a boring portfolio. Both the company and the candidate wasted time talking to the agency because one way or another, we had to figure out most things for ourselves. The most interesting facts were those off the record.
2. One agency used to save these recommendations as DOCX files. I work on Ubuntu and LibreOffice did not properly render those files, throwing some contents away from the page. I had to switch to Windows, launch Microsoft Word, prepare myself a PDF file and switch back. Eventually, the HR guys learned how to create PDF themselves. What a relief.
3. The same agency stripped my recruitment task. It originally was a GitHub repo which consisted of some important files in the root directory (`composer.json`, `docker-compose.yml`, `phpunit.xml.dist` and – most important – `README.md`!) and a `tests` directory. You had to write a simple implementation which passed all of my tests. I surprisingly found out that all candidates sent by that agency rewrote `composer.json` on their own. I asked them: “Why would you do that? The whole autoloader was defined there!.” It turned out that the agency sent the candidates only the tests directory. They did not send the full repo, of course – the candidate could find out what company made that task. The target company’s name is top secret in the beginning – and I recklessly used it as a vendor part of PSR-4 namespaces.
4. The situation with missing `composer.json` involved also one candidate which was so tired of endless meetings with an agency that – after learning what company is he applying to – he declined to cooperate with the agency and sent his resume directly to us. I didn’t know that story and I was surprised to see that he already solved our task.
5. Speaking of endless meetings – that’s the thing I always hear if someone decides to share his or her experience with an HR agency. When you see a fancy job ad like “For our client, a market leader…” and you send your resume, you’re invited to an entry interview in the agency HQ. Then you receive our task to do at home. Then they invite you to another meeting… but not with us! If that meeting succeeds, you eventually make your way to our facility. You go there and probably hear the same questions you’ve already answered in the previous meetings, because the incompetent agency did not properly sum up your answers and we don’t have enough data about you.
6. Sometimes we don’t receive a full resume, but only a solved task (kind of a blind recruitment). Sometimes we receive a resume with a surname washed out. Sometimes we receive only the recommendation and not the original resume.
7. Sometimes a candidate is forced to visit the target company together with a guy from HR, who ensures that we don’t make a deal behind their back and we don’t abuse you. To me it’s like coming to an interview with your dad! He’s a man-in-the-middle. It’s not a deal between an employee and an employer. Fortunately, your “dad” stays only for a “soft interview” and he walks away during the tech interview (which I lead). He goes for a coffee and waits there until we’re done. So basically he gets paid for drinking coffee in our office.
8. It’s funny to see how an extravert, upbeat HR guy brings a terrified candidate to the meeting. At the first glance I would hire the HR guy, not a shy, scared dev.
9. My coworker received an offer for my position from an agency when I decided to leave the company.
10. My boss received an offer from an agency to hire a colleague who was just about to leave the company.
11. When you get hired and work for a while, the agency calls you from time to time to ask how is it going. That’s what my coworkers told me. As an employee, I would find this annoying.

I have no way to convince my soon-to-be-ex employer to stop wasting money with HR agencies (or invest in the right one). But I still wonder why good devs in Poland respond to the job ads posted by agencies where you don’t have a target company’s name disclosed. The salary range is not specified either. If I invest my time talking to the HR guy, I would like to know in advance who am I going to work for and what salary I can expect!

## Carving my own path

Like I said in the beginning, I have never looked for a job through an agency. I have some basic googling skills, but let me show you a brief story of my career which provides some more tips:

1. I met my first employer during high school. It was a local news company. My friend had already worked for them and he asked if I could make some photos at local events. Still being a student, I started my humble photography career in my spare time. After my high school exams (Polish matura) and before going to university, I got a permanent job offer. They discovered I can code PHP for food, so they hired me to work on their new website during the day AND make photos in the evenings. Weird setup, but this job gave me a lot of life experience.
2. After five years, I met my future girlfriend. I decided to relocate to Gdańsk which is ~300 km away from my hometown. I started browsing pracuj.pl, a Polish job board and soon I found an offer from an education company. I sent my resume, did a recruitment task and after three weeks I got an offer! I’ve been working there for over four years. During that time, my salary has doubled and I went from a mid developer to a team leader.
3. Me and my girlfriend decided to change our lifestyles and travel more. I needed a full remote job. I remember meeting a nice software house at a PHPers conference in Poznań. I sent my resume. Unfortunately they just stopped their recruitment process at a time. But after a month I received an e-mail inviting me to a new process, which I passed successfully. I had three interviews via phone and Skype: entry, tech and soft skills. I really liked all those interviews. One of my interviewers said he saw my PHPers presentation about database optimization and he already knew my name. I got the offer with a salary which allows me to live decently, buy more music equipment (oh yeah!) and even save money for retirement.

As you can see, my ~10 years career as a developer did not include any deals with any recruitment agencies. It is important to say that I’m not an easy-going, upbeat and extravert person. I’m not good at getting my foot in the door, but at least I’ve learned how to create my resume properly, find a possible employer and make him interested in my offer. I’ve spent a lot of time browsing Polish [nofluffjobs board](https://nofluffjobs.com/) where all the offers are plain and simple, with salary specified upfront.

What’s more to do? I guess I should visit more conferences and give more talks, so that people will know my name. I should write more blog posts and possibly contribute to Open Source. That way I’m going to develop my personal brand and hopefully do my business stuff without the HR agencies.

If you work in an HR/talent agency, please improve your skills. I know that IT headhunting is very hard and it might be really frustrating to abuse LinkedIn for the whole day just to receive a bunch of rude replies (or no replies at all). But if you want to do a good job as a headhunter, you need to understand how candidates and companies behave and what they really need. We’re all here to make business happen, right?
