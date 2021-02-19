---
layout: post
title:  "Doing Java the bad way: loose typing"
date:   2020-01-03 17:00:00 +0100
permalink: /2020/01/03/doing-java-the-bad-way-loose-typing/
description: Overusing HashMaps in Java web applications is an antipattern that makes Java work like PHP - in a bad way.
tags:
  - clean code
  - java
  - php
---

In the late 90s, people loved PHP because it was so easy to use. And one of the reasons for that was [loose typing](https://www.php.net/manual/phpfi2.php#starting). You could quickly send any data to the browser without caring about [data types](https://www.php.net/manual/en/language.types.type-juggling.php).

This became a problem when developers started to build bigger and bigger applications with PHP. Strict typing became an important feature – first introduced in [HHVM](http://hhvm.com/) and later adopted by PHP 7. However, there were countless discussions on PHP internals list on whether to enforce strict typing or not. In the end, [strict typing became a feature on request](https://www.php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict). And it didn’t work for variables or class properties (the latter will change with PHP 7.4).

What I always liked about Java was *strict typing everywhere* from day one. Java code *screams* to me, like “hey, I got a `product` object of type `Product` and this object cannot be anything else by accident”.

Later I saw that **you can make a big mess with every technology, even Java.**

## Everything is an Object

Every Java object inherits from the `Object` type. And you can even [create hashmaps](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) of anything that derives from `Object`. So these maps can even contain further maps of every possible creation.

It is sometimes tempting to code Java the PHP way. It’s quick. You can have one box for every kind of stuff. But then it becomes a nightmare.

**First warning sign of an antipattern:**

```java
Map output = new HashMap<>();
output.put("stuff", stuff);
output.put("otherStuff", otherStuff);
response.setData(output);
```

**Second warning sign:**

```java
public void setData(Object data) {
    // Oh, so I can put any data here...?
}
```

**Third warning sign – the tests:**

```java
Map output = (Map) response.getData();
// now we get an "unchecked warning" and we have no idea what's inside "output"
```

Try to answer this tricky question first: *What does my web app do?* You could say: *Well, it retrieves data from the database and sends them to the user.*

Wrong answer. How about this one: *My app retrieves products and orders from the database and sends them to the user.*

If you put it that way, you’ll begin to understand that you need classes like `Product`, `ProductsList`, `ProductsListRequest`, `ProductsListResponse` and so on. Well, maybe not all of them (don’t over-engineer!), but at least don’t put everything possible into the data bins.

Every **HTTP request and response** in your application must have a specified **structure** anyway, so try to **formalize it** by creating proper classes.
