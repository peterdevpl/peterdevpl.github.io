---
layout: post
title:  "How a legacy code is made"
date:   2018-07-13 17:00:00 +0100
description: An essay about struggling with a legacy code, splitting the codebase into layers using hexagonal architecture and Domain-Driven Design.
excerpt: I received an apparently easy task to do. It should take maybe 15 minutes. Or maybe I should not even do it at all. Salespeople should have a reporting feature for that. However, it turned out that a mechanism to fetch products with prices was strictly tangled with the way these items were presented in the shop.
permalink: /2018/07/13/how-a-legacy-code-is-made/
tags:
  - architecture
  - domain-driven design
  - legacy
  - php
---

> “The (…) problem with legacy assets is how difficult and risky they are to invoke. They were developed with a particular feature-set in mind, and are highly coupled to those features. This makes it hard to access what you want directly without going through layers designed for very different purposes. The data you want is nearly always intertwined with other data and functionality, so that it is hard to detach just the part you want without dragging along an endless tangle of interactions and associations.”
>
> Eric Evans, “[Getting started with DDD when surrounded by legacy systems](https://domainlanguage.com/ddd/surrounded-by-legacy-software/)“, 2013

I received an apparently easy task to do. A head of sales wanted to receive a list of all products sold in one of the departments. She expected a table with just three columns: product name, default rebate name and rebated price. Almost every product had some rebate attached to attract clients.

This task should take maybe 15 minutes. Or maybe I should not even do it at all. Salespeople should have a reporting feature for that. However, it turned out that a mechanism to fetch products with prices was strictly tangled with the way these items were presented in the shop. I couldn’t retrieve all the products at once because I was forced to do it page by page (maximum 50 products per page). The rebate system worked only for a currently logged in user and everyone could have different rebates based on the account settings. I could not easily simulate rebates on another accounts. Eventually, preparing a rather simple products summary took three hours of writing a very dirty code…

The shop was custom-made to meet strict specifications oriented towards an ordinary web browser. From an end user’s standpoint, a shop should have a paginated list of products, a shopping cart, rebates info etc. Just some standard e-shop features. No one thought about additional use cases that might show up some day – like salespeople asking for summaries and reports. No one thought that some day we would need to share data through an API endpoint. Deadlines were coming after us. All developers just wanted to get it done according to specs. A legacy, proprietary PHP framework did not make it any easier.

That way, we created lots of code which became “legacy” and inefficient just after half a year past deployment.

## Separating code responsibilities into layers

To handle such different requirements, a multi-tier/multi-layer and [hexagonal architectures](https://leanpub.com/ddd-in-php/read#appendix-a-hexagonal-architecture) were invented. Sometimes, simple CRUDs shown in framework’s documentation are not enough; complex applications need a clear separation between a business/domain logic, infrastructure and presentation. We create a network of loosely coupled modules which, thanks to interfaces and [dependency injection](https://symfony.com/doc/current/service_container.html), can be easily switched and replaced. This approach has a lot of pros:

* Every layer can be, to some extent, developed independently. Because of abstraction, the business logic does not depend on a particular way of presentation, page rendering, data storage etc.
* Every layer can have a separate set of automatic tests.
* While editing the code of one layer, we are not distracted by the code of other layers.
* The same data can be presented in multiple ways. We just attach a different presentation layer implementation (HTML, PDF, JSON API, CLI…).
* Business rules are clear. We don’t need to look for them in strange places, like templates.

> “Allow an application to equally be driven by users, programs, automated test or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases.”
>
> Alistair Cockburn, “[Hexagonal architecture](http://alistair.cockburn.us/Hexagonal+architecture)“

In practice, using a multi-layered, hexagonal architecture with loose coupling needs a lot of experience. I discovered that it’s really hard to convince a team to talk about abstract models. People tend to ask about the details very early, like database, graphics – and there’s nothing wrong about it. People need to imagine the final effect, a real use case – it makes them easier to understand the project.

During project discussions, I suggest not to avoid speaking about implementation. However, we should strive for a transparent and fluent code. It’s worth negotiating some additional time with the client, so that we can create a good project that will save him maintenance costs in the future.
