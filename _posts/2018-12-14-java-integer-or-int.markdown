---
layout: post
title:  "Java: Integer or int?"
date:   2018-12-14 17:00:00 +0100
permalink: /2018/12/14/java-integer-or-int/
tags:
  - java
  - npe
  - sql
---

When I first saw primitive types like `int` or `boolean` mixed up with classes, I was very tempted to convert all primitives into `Integer`, `Boolean` and so on to maintain a clear coding style. But reading articles on the Internet and IntelliJ hints stopped me from doing such a stupid thing.

I read this wonderful rule of thumb:

> Don’t create unnecessary objects.

Primivites always occupy less memory than objects, which was clearly described [in this article on primitives](https://www.baeldung.com/java-primitives-vs-objects). Maybe we don’t usually care about the memory usage, but if we process big amounts of data, every byte can count. Moreover, accessing primitives is faster because they are stored on the stack, not on the heap.

It’s also important to remember that a variable pointing to an `Integer` or `Boolean` object can be `null`. A primitive can’t. This difference matters for example when you retrieve data from an SQL database which allows NULLs. If you try to assign `null` to a primitive, you’ll get a Null Pointer Exception. That’s a bug I noticed and fixed in a production system. So I always try to restrict the possibility to use NULLs in the database and in the code unless `null` has a real business meaning.
