---
layout: post
title:  "NPE: Converting a list of objects to a map"
date:   2018-11-30 17:00:00 +0100
permalink: /2018/11/30/npe-converting-a-list-of-objects-to-a-map/
tags:
  - java
  - npe
---

Like every Java developer, I had a fair amount of Null Pointer Exceptions in my career. That’s why I decided to describe real-world examples I had to fix.

Today I will show you how to get an NPE while retrieving records from a database and creating a **map** based on these records, using streams and collectors introduced in **Java 8**.

Let’s say we have a `Setting` class:

```java
public class Setting {
     private final int accountId;
     private final String type;
     private final String value;

     public Setting(int accountId, String type, String value) {
         this.accountId = accountId;
         this.type = type;
         this.value = value;
     }

     public String type() {
         return type;
     }

     public String value() {
         return value;
     }
}
```

Now, in another part of the system we will retrieve `Setting` objects from a database. We haven’t noticed that the SQL table allows NULL in the value column. Let’s pretend our list of settings looks like that:

```java
List<Setting> settings = new ArrayList<>();
settings.add(new Setting(1, "site_title", "My blog"));
settings.add(new Setting(1, "site_description", "A place with fresh ideas"));
settings.add(new Setting(1, "site_copyright", null));
```

We want to easily access settings by names, so we create a **map** using a **stream**:

```java
Map<String, String> settingsMap = settings
     .stream()
     .collect(Collectors.toMap(Setting::type, Setting::value));
```

**Bang!** We’ve got a NPE here because one of the values used to create a map is `null`. It turns out that `Collectors.toMap()` does not handle NULLs.

Let’s think: **do we really need a NULL in this case?** I guess not. A NULL value means “no data” or “unknown value”, which is the same as if the setting did not exist. We should set the `value` column as `NOT NULL`. We can additionally filter the `settings` list:

```java
Map<String, String> settingsMap = settings
    .stream()
    .filter(setting -> null != setting.value())
    .collect(Collectors.toMap(Setting::type, Setting::value));
```

A NULL would make sense for example if we wanted to create a map between people’s names and their ages. If we don’t know someone’s age, then a `null` seems reasonable. We would have then to create a map in a different way.

You can have a further read on [StackOverflow](https://stackoverflow.com/questions/42546950/use-java-8-streams-to-transform-a-map-with-nulls/42559100). Alternative ways to convert a list to a map can be found on [Baeldung](https://www.baeldung.com/java-list-to-map).
