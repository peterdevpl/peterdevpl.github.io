---
layout: post
title:  "Why you should disable wildcard imports in IntelliJ IDEA"
date:   2019-10-18 17:00:00 +0100
description: The default behavior of IntelliJ IDEA is to replace multiple class imports from a package with an asterisk. At my team, we decided to avoid that behavior. Why?
excerpt: The default behavior of IntelliJ IDEA is to replace multiple class imports from a package with an asterisk. At my team, we decided to avoid that behavior. Why?
permalink: /why-you-should-avoid-star-imports-in-intellij-idea/
redirect_from: /2019/10/18/why-you-should-avoid-star-imports-in-intellij-idea/
tags:
  - clean code
  - java
---

The default behavior of IntelliJ IDEA is to replace multiple class imports from a package with a star/wildcard/asterisk. So for example when you’ve got:

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;
```

…and you try to import another class from `java.util`, everything is merged into one mass import: `java.util.*`.

At my team, we decided to turn that behavior off. Why and how?

1. **It is bad for version control.*** Suppose one developer is using Eclipse and just adding another line to the imports. Then another developer is editing the same file and merging all the imports. We’ve got a merge conflict here! Also, with star imports, it’s harder to track what has been added to the local file namespace.
2. **It can lead to unnecessary naming collisions.** There can be a lot of User classes in different packages. By importing everything from a package we can accidentally import something we do not want. Also, when a dependent package is upgraded, it might pollute our namespace with new imports that we’re not even aware of.

**IntelliJ does not have a way to turn off star imports, but it has a threshold option. So everyone in the team set a ridiculously high threshold (like 500):**

![Here you can prevent IntelliJ from merging your imports](/assets/intellij_star_import_threshold.png)

**Another problem is folding imports by default.** I believe that the imports section is actually a very important piece of code and it should not be folded. We need to be aware of what’s going on there, so we jumped into `Settings` > `Editor` > `General` > `Code Folding` and turned `Imports` off.

## Further reading

[https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html#disable-wildcard-imports](https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html#disable-wildcard-imports)
