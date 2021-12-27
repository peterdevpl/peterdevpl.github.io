---
layout: post
title:  "Secure generation of random IDs and passwords in Java"
date:   2021-01-10 17:00:00 +0100
description: "Tools which provide safe, unpredictable random numbers and strings in your Java application: SecureRandom, Apache Commons and Passay."
permalink: /secure-generation-of-random-ids-and-passwords/
redirect_from: /2021/01/10/secure-generation-of-random-ids-and-passwords/
tags:
  - java
  - security
---

The Apache Commons Lang library has a handy set of random string generators, enclosed inside the `RandomStringUtils` class. However, these are not cryptographically secure generators by default, which can trigger warnings in platforms like Veracode (for example [CWE-331: Insufficient Entropy](https://cwe.mitre.org/data/definitions/331.html)).

It’s even more important when you think what the random strings are used for. Most of the time these are some session, token or debugging identifiers, or even passwords. Such strings shouldn’t be predictable.

The default `java.util.Random` implementation is not cryptographically secure, and yet it is used by default in shorthand `RandomStringUtils` methods. However it is possible to pass a custom generator as the last argument, for example `java.security.SecureRandom`:

```java
final SecureRandom random = new SecureRandom();
final String id = RandomStringUtils.random(10, 0, 0, true, true, null, random);
System.out.println(id);  // prints 10 random alphanumeric characters
```

A more sophisticated yet cleaner way might be to use `RandomStringGenerator` from Apache Commons Text together with `SecureTextRandomProvider` from Apache Syncope. Unfortunately, the latter class was removed in Syncope 2.1 and I couldn’t find any alternative.

Looks like Apache doesn’t like providing cryptographically secure random generators or even interfaces for them. The [Apache Commons Random Numbers Generators documentation](https://commons.apache.org/proper/commons-rng/) says: *“The current design has made no provision for features generally needed for cryptography applications (e.g. strong unpredictability).”*

One more library worth checking out is [Passay](http://www.passay.org/). Its primary responsibility is to maintain a company’s password policy, and the library can be also used to generate random passwords according to the company rules. Of course you can provide `SecureRandom` as the source of randomness:

```java
final SecureRandom random = new SecureRandom();
final PasswordGenerator generator = new PasswordGenerator(random);
final CharacterRule alphabet = new CharacterRule(EnglishCharacterData.Alphabetical);
final CharacterRule digits = new CharacterRule(EnglishCharacterData.Digit);
final String id = generator.generatePassword(10, alphabet, digits);
System.out.println(id);
```

This was just a basic example; Passay accepts more complex rules, for example a minimum number of letters, digits or special characters in a password.

## Final thoughts

Use `SecureRandom` whenever you need to generate a random string.

Never use regular expressions to validate passwords against the company policy. Just don’t. Or you will end up with monsters like this (true story):

```java
String PASSWORD_REGEX = "^[A-Za-z0-9!@#$%^&*()\-_=+:;'\"<>,.\\]{8,}$";
```
