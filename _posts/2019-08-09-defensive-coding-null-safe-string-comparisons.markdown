---
layout: post
title:  "Defensive coding: Null-safe string comparisons"
date:   2019-08-09 17:00:00 +0100
permalink: /2019/08/09/defensive-coding-null-safe-string-comparisons/
description: Avoiding Null Pointer Exception during string comparison in Java
tags:
  - defensive coding
  - java
---

One of the most valuable tips I received why having my Java code reviewed was to change this code:

```java
if (merchant.getSalesChannel().equals("Mexico")) {
    /* … */
}
```

to this:

```java
if ("Mexico".equals(merchant.getSalesChannel()) {
    /* … */
}
```

Everything in Java is an object. Thus, a simple string literal immediately creates a new object. We are sure that a literal is not null.

The other part of the equation might surprise us with a null. Who knows what that getter might return? Most likely it is some value from the database. Are we sure that the value was properly retrieved? Are we sure there’s a `NOT NULL` constraint on the database column? We can never be sure.

So if we want to compare a variable to a string literal, let’s use the null-safe style proposed in the second listing above.

### Further reading:

[https://stackoverflow.com/questions/43409500/compare-strings-avoiding-nullpointerexception/43409501](https://stackoverflow.com/questions/43409500/compare-strings-avoiding-nullpointerexception/43409501)

[https://softwareengineering.stackexchange.com/questions/313673/is-use-abc-equalsmystring-instead-of-mystring-equalsabc-to-avoid-null-p](https://softwareengineering.stackexchange.com/questions/313673/is-use-abc-equalsmystring-instead-of-mystring-equalsabc-to-avoid-null-p)

[https://www.javacodegeeks.com/2012/06/avoid-null-pointer-exception-in-java.html](https://www.javacodegeeks.com/2012/06/avoid-null-pointer-exception-in-java.html)
