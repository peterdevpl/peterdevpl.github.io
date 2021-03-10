---
layout: post
title:  "Too much REST will harm you: don’t blindly follow it!"
date:   2021-03-10 20:00:00 +0100
permalink: /2021/03/10/too-much-rest-will-harm-you-dont-blindly-follow-it/
image: /assets/rest_example.png
description: Always treat a URL like public data. Although REST recommends a set of web architecture best practices, you need to take additional measures to prevent sensitive data exposure.
excerpt: "REST, or Representational State Transfer, is a set of web architecture best practices. Perhaps it is best known for associating resources and actions in order to create clean API interfaces. Although REST works perfectly fine in most situations, I will show you how it can cause security issues where security matters most: the payment industry."
tags:
  - security
---

> The screenshot above shows an example REST API described by Swagger (from [petstore.swagger.io](https://petstore.swagger.io/)).

REST, or Representational State Transfer, is a set of web architecture best practices. Perhaps it is best known for associating resources and actions in order to create clean API interfaces. Although REST works perfectly fine in most situations, I will show you how it can cause security issues where security matters most: the payment industry.

[The principles of REST](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) were described in 2000 by Roy Fielding in his doctoral dissertation. He also co-founded the Apache HTTP Server project and he chaired the Apache Software Foundation. Who am I to disagree with such an accomplished man?

I’ve been working as a developer in the payment industry for two years and I’ll tell you one thing: this is different from most software development jobs. You can get away with a lot of bad behaviors in other industries, but here they are just unacceptable. For example, I know a case of a person who was fired from a payment company after editing production database without permission.

**Security and reliability** are obviously the apple of the payment industry’s eye. Companies have to pass external security audits which confirm compliance to several standards like the [Payment Card Industry Data Security Standard](https://www.pcisecuritystandards.org/). Otherwise, if hackers compromise their systems, these companies might go out of business.

When you start working in a high profile industry like this, suddenly you have to face challenges you were previously unaware of.

## A verb and a resource: the essence of REST?

What Roy Fielding basically did was to promote best practices for the World Wide Web architecture and “identify architectural mismatches.” This became important as he witnessed multiple engineers around the world rapidly pushing the web into multiple directions, often making design mistakes or deviating from initial concepts.

As we know, the web consists of “resources” identified by their “locators” (URLs). **A locator identifies a resource and tells you where you can find it.** You can perform several tasks with these resources. The most common task is to GET it. You can also PUT a resource on the server, or even DELETE it. A POST method is commonly used to submit forms and also binary files, JSON or other data structures.

This simple and clever concept allows you to build a clean and understandable interface of a web service. Let’s say we have a blogging platform:

* `GET /article/too-much-rest` will simply retrieve the article identified by a string “too-much-rest.”
* `PUT /article/too-much-rest` will either create a new article or update an existing one. Server expects the request body to contain article contents.
* `DELETE /article/too-much-rest` will remove the article “too-much-rest” if it exists.

As I said before, a URL serves two purposes. First, it identifies a resource. The string `article/too-much-rest` appears in all above URLs and it suggests that there is only one such resource on the server. Additionaly, an absolute URL like `https://example.com/article/too-much-rest` will also tell us where the article is stored and what protocol is used to communicate with that server.

Mr. Fielding has put a lot of pressure on [using proper verbs for certain actions](https://roy.gbiv.com/untangled/2009/it-is-okay-to-use-post), for example:

> It isn’t RESTful to use POST for information retrieval when that information corresponds to a potential resource, because that usage prevents safe reusability and the network-effect of having a URI.

## Avoid exposing sensitive data in URLs!

So far we’ve discussed an example public API for a blogging platform. Nothing sensitive there. Blogging *is* public most of the time.

Now, imagine you are developing a top-secret system which handles customers’ names, their account numbers, passwords, SSNs, transaction IDs, session IDs, and so on. Imagine billions of dollars flowing through that system.

You should never create even an internal, private API like this:

* `GET /customers/social-security-number`
* `GET /customers?name=John&surname=Doe&city=London`
* `POST /customers/data?sessionId=1234`

The reason is simple. **Always treat a URL like public data because:**

* It is stored in browser’s history, sent over network to synchronize your account between devices, submitted to search engines, and so on.
* It can be copy-pasted over various communication platforms. Especially if the URL is long, it’s easy to ignore sensitive data it contains.
* It can be sent in a `Referer` header to other sites.
* It is logged by multiple network devices, servers and proxies. These logs can be aggregated by third-parties, exchanged over insecure channels, etc.

## Pentesters are going to report this

I once had a lot of work redesigning a robust system just because a pentester discovered sensitive data sent over GET requests. Earlier, a developer simply wanted to design a “proper” REST API, so they put customers’ data in the URLs.

The Common Weakness Enumeration (CWE) calls this vulnerability [“Use of GET Request Method With Sensitive Query Strings”](https://cwe.mitre.org/data/definitions/598.html):

> The query string can be saved in the browser’s history, passed through Referers to other web sites, stored in web logs, or otherwise recorded in other sources. (…) At a minimum, attackers can garner information from query strings that can be utilized in escalating their method of attack.

Exposure of sensitive data is also listed as #3 in the [OWASP Top Ten](https://owasp.org/www-project-top-ten/) Web Application Security Risks. [URLs are pointed out as one of the ways data can be exposed](https://owasp.org/www-community/vulnerabilities/Information_exposure_through_query_strings_in_url).

## How to protect your system?

Analyze what data the system processes and how. Limit the amount of processed data to the minimum. Don’t send more fields than needed, “just in case someone needs them in the future.”

Use the POST method to send sensitive, private data inside a request body. This is important even if the requests flow only inside an internal company network.

You can even perform “POST redirections” if needed. Instead of sending a mere `Location` header, prepare a HTML form and submit it automatically by JavaScript. [Even IFRAMEs can be loaded with POST](https://css-tricks.com/snippets/html/post-data-to-an-iframe/). Many payment platforms work this way.

Another solution is to encrypt URL parameters. However, there are [different opinions on URL encryption](https://paragonie.com/blog/2015/09/comprehensive-guide-url-parameter-encryption-in-php). This isn’t an easy task because encryption algorithms get cracked sooner or later. There are many nuances to think about, like padding. Consider if the benefits are worth the effort.

## “Form follows function”

Remember that in the 1990s, when the World Wide Web was born, its creators dreamed about a publicly accessible repository of knowledge and services — for everyone. On the contrary, most modern industries require utmost privacy and security.

I believe that Roy Fielding and other wise creators of the web standards did an awesome job. However, please keep in mind the web is such a dynamic environment that anything can happen. Always try to choose right tools for the job. Don’t stick to buzzwords.

It is important to realize that the web still grows rapidly all around the world and many of its core technologies are used in a way not predicted by their makers. It is your responsibility to use them wisely.

*Article originally published on [Medium](https://peterdevpl.medium.com/too-much-rest-will-harm-you-dont-blindly-follow-it-cc994a1c0df2)*
