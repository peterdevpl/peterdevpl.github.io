---
layout: post
title:  "8 Programming Myths That Impede Your Career"
date:   2022-04-18 12:00:00 +0100
description: "Common beliefs and misconceptions can impede your software development career. Here's my guide on how to avoid fighting for a lost cause."
excerpt: "During 14 years of my software development career, I've seen - and was a victim of - numerous myths and fads of the IT industry. Common beliefs and misconceptions often impeded my career because I wasted energy on activities that did not bring the expected benefits. Here's my guide on how to avoid fighting for a lost cause as a software developer."
permalink: /programming-myths-that-impede-software-development-career/
tags:
  - management
  - soft skills
  - team leader
---

During 14 years of my software development career, I've seen - and was a victim of - numerous myths and fads of the IT industry. Common beliefs and misconceptions often impeded my career because I wasted energy on activities that did not bring the expected benefits.

Here's my guide on how to avoid fighting for a lost cause as a software developer.

## We must have Scrum

Most companies have management issues, and once a software developer experiences problems, they believe there must be a clever solution. Some silver bullet, some magic pill to end all pain.

Scrum is often depicted as such a magic wand, but a struggle to adopt it may become even more frustrating than the initial problems. We think that Scrum is a perfect solution, so we assume that it's all our fault that we can't get it work.

Leading people is a lot more than telling them what to do. Building an organization's culture is a lot more than setting up Jira.

Although Scrum is a "lightweight framework", it still imposes rules that some organizations will be unable to adhere to:
1. We work in constant timeframes ("sprints") and try not to interrupt that work.
2. We have meetings (daily, review, planning, retro) in constant time and place.

If the company does not respect these two rules, Scrum won't work. However, there are so many other things you can do to improve the management culture:
* Establish a better feedback loop with business people. You need to know each other's problems. You need to know how your technical solutions perform in real life. They need to know what it takes to build an IT system.
* Discuss business priorities. Inform your client that you can't do everything at once. Explain that multitasking causes bad performance.
* Introduce high quality standards in your team. Static code analysis, tests, CI/CD, any kinds of automation.
* Insist on transparency and good communication.
* Integrate your team. Go out with them, have fun, get to know each other.

## 100% code coverage

Code coverage is a metric that defines a percentage of lines of code executed during tests. It is a common belief that higher test coverage means better quality.

How about this example:

```java
class CalculatorTest {
  @Test
  void shouldPerformAddition() {
    int result = calculator.add(2, 2);
    
    assertTrue(true);
  }
}
```

This test verifies only the fact that the code has been executed without errors. But there is so much more to do! We need to check the logic, verify calculations, find edge cases.

In general, the more complex is the code, the more test cases you should have. A single line of code may be covered by multiple tests.

If your team aims only at the highest line coverage possible, it might:
* give you a false sense of safety while in fact, the tests may pass serious bugs, and
* encourage people to write phony assertions just to boost the overall metric.

A great tool to measure quality of tests is *mutational testing.* Programs like InfectionPHP generate different versions of your production code, for example by altering conditions, removing lines, and so on. If tests do not fail despite these changes, it means they don't catch enough bugs (mutants). Usually these problems get fixed by writing more precise test cases.

## Rewrite is necessary

Developers love greenfield projects because they treat them as playgrounds to try all the fancy techniques, tools and frameworks they crave for.

While maintaining an old and messy project, there's usually a temptation to rewrite it from scratch. "This time we'll do it better," everyone thinks. This is rarely the truth.

Apart from obvious technical circumstances, like Adobe Flash being retired, a rewrite causes more harm than good. Martin Fowler in his book "Refactoring" tells a story of a software project gone down because of a fatal attempt to rewrite everything. The project was late, over budget and didn't work properly.

My first action when dealing with a legacy project is to write proper tests, which are often missing. I want to understand the system's behavior, including all hidden behaviors and side effects. With a good set of unit, integration, API and E2E tests I can proceed to refactor the most annoying parts of the system. Tests make me confident that I don't break any of the existing behaviors that users actually rely on.

There are other efficient strategies to deal with legacy systems that do not involve a major rewrite: Strangler Pattern, Anti-Corruption Layer, Facade. All of them assume that you start building new modules step by step, but still route traffic to the old code. When a new module is ready, you just switch traffic.

Instead of conducting a costly rewrite that takes months to complete, it's better to improve the project step by step and have a tight feedback loop. You can release a small fix every week and see if it's working properly and how it contributes to the overall project quality.

## Sophisticated architecture is cool

A lot of engineers believe that complicated things are more professional. When they master a difficult technique, they feel an urge to prove themselves and use the newly acquired skill in real life.

This goes along with trying to build the most flexible, dynamic and abstract system possible. As the source code is split into more and more layers of abstraction, it becomes more difficult to understand and maintain.

The urge to complicate software design is caused by:
* **The fear of legacy code.** Developers traumatized by old, messy codebases are trying to avoid them so hard by using fancy design patterns that they're actually making a new mess that only looks clever on the outside.
* **The fear of change.** Developers notoriously ask this question: "What if business requests a change?". This often goes in pair with the business imposing strict deadlines. Developers try to anticipate these requests by building a "flexible" system, but the development delays because of all the crazy tricks in the code.

I have two principles that help me overcome those fears: Just In Time and Keep It Simple, Stupid. I don't have to build an empire on day one. Let's start simple.

There has to be a balance. Not every project requires Domain-Driven Design, or CQRS. Not every project requires an ORM. Not every project requires a ton of interfaces, abstractions, layers, providers, resolvers, adapters, or whatever fancy design pattern you love.

When I'm forced by business people to deliver working software fast, I always tell them: okay, I can cut corners, but the future changes will take more time. I warn them, and it's fine. Business sometimes wants just to validate an idea, or quickly solve an issue. If you're patient enough, they will eventually understand the technical consequences.

Also, remember the more sophisticated your architecture is, the more difficult it will be to onboard new developers.

## Must have the latest version

Most software projects depend on external libraries and tools. It can be satisfying to upgrade to all new versions available, but how can you be sure that it's better, or at least still works?

Despite semantic versioning promises, and all the effort that went into Quality Assurance, even the most professional software vendors make mistakes. Even a tiny update from version 1.2.0 to 1.2.1 can introduce bugs.

On the other hand, regular updates are important due to security issues being discovered. It's also cool to work on recent software and easier to attract new talents.

You can save yourself from trouble and make updates easier by implementing integration and E2E tests.

## Must follow all the trends

The so-called "Hype-Driven Development" (or ironically, "Resume-Driven Development") has many victims. There can be a strong neophyte effect after reading a popular book or attending a tech conference. People think that their workplace can be improved only by applying all the recent discoveries.

Somewhere between 2015 and 2018, there was a huge hype on microservices. Conference speakers claimed that we should split old monoliths into flexible microservices, just because Netflix does this. They didn't warn about all the additional problems caused by the new approach: performance, stability, data separation, and so on. Several years later there were voices saying that *microservices are not for everyone* and you should consider a *modular monolith.*

It's good to know what happens in the industry, but you shouldn't adopt every new buzzword. Carefully analyze whether the new solution fits your project and organization.

## We don't need meetings

Business people focus on meeting people, talking to them, building relationships and a network. Developers love to focus on the code. They depict every meeting as an interruption, "not work."

How many times did you hear a meeting being concluded with these words: "Ok, let's go back to work." Was that meeting not work? Of course it was, but for developers, only coding feels like "real work." This is a mistake.

Developing software is a team effort, and to build a team (and a product), you have to talk to each other.

If you feel overwhelmed by meetings, possible solutions include:
1. Having a clear goal and agenda for every meeting. If you receive an invitation without these things specified, ask for details or deny.
2. Putting all the meetings (like Scrum ceremonies) in one day. This works really well for my team.
3. Adding "Focus time" or similar items in your calendar, so that other people know you're busy. You have a right to go offline from time to time!
4. Picking a moderator for every meeting. That person is responsible for making sure a meeting is effective and comfortable. It can be a Scrum Master, but doesn't have to.
5. Utilizing every tool possible to make communication better: webcams, Miro, Notion, G-Suite. Instead of just talking, make everyone collaborate on a document, diagram, drawing.

## Business doesn't understand us

It's common among developers to think that the "ordinary" business people don't understand nor appreciate how "clever" the developers are. Whether business is requesting crazy features, imposing deadlines, or just complaining about broken software - I can often hear this voice inside development teams: "oh, they don't understand."

Business people focus on getting clients and making money. Developers focus on technology. It's important to both parties to explain difficult topics to each other. Why is the business doing all these pivots? Why are the developers talking about a rewrite again? Just talk to each other, and it will already solve a lot of problems.

Another important thing to do is to get out of your room and gain a wider perspective. When you put so much effort into solving a small coding problem, it can be totally insignificant from an overall company's standpoint. Relax and focus on something that matters more.
