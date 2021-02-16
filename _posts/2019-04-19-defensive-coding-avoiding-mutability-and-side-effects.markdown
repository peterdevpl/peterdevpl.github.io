---
layout: post
title:  "Defensive coding: Avoiding mutability and side effects"
date:   2019-04-19 17:00:00 +0100
permalink: /2019/04/19/defensive-coding-avoiding-mutability-and-side-effects/
tags:
  - defensive coding
---

Are you tired of fixing the same bugs on and on in a huge system developed by a multitude of developers? I guess it’s time to introduce some practices of Defensive Programming. It is an approach to improve software quality by reducing number of possible bugs and mistakes and making the code more predictable and comprehensible.

Before we continue, I advise that you get familiar with NASA’s [Power of 10 Rules](http://spinroot.com/gerard/pdf/P10.pdf), designed for developing

You can also read about [poka-yoke](https://en.wikipedia.org/wiki/Poka-yoke) approach to design things in a way that prevents possible misuses. An example from a real world is a SIM card that can be inserted in only one position. Now think how many times, as a developer, you were misled by someone else’s code? How many times have you used wrong function, wrong variable, wrong argument, wrong format?

Writing reliable code is an art of avoiding mistakes. Let’s see how to do that.

## Avoid mutability and side effects

How many variables are there in your code? How many of them are not used anymore? How many of them have misleading names?

Every variable can affect the behavior of your program (well, that’s what they’re designed for). Variables are meant to have different values over runtime. Are you sure you’re controlling all of them properly?

Here are some basic rules for variable handling:

1. **Do you really need a variable?** Maybe a constant will be enough? In JavaScript, use `const` instead of `let`. In Java, use `final` to indicate the value can be assigned only once.
2. **Use the strictest scope possible. Avoid global variables.**
3. **Avoid modifying object state and global state if not necessary.** A function that modifies anything outside the scope of itself is creating a **side effect**. Side effects often cause unpredictable bugs and make testing more difficult. Even if you just poke the objects passed as function arguments, it is a side effect. At least be aware of the consequences.
4. For simple values like date/time, **use immutable value objects**. For example, if you want to add 2 months to an existing date, create a new result object instead of modifying the existing one.
5. **Use precise and meaningful naming**, not just `i`, `j`, `k` or `temp`. How can other developer know which variable is responsible for what? On the other hand, take context into account. Avoid lengthy names with unnecessary prefixes, like `bigFancyModuleUser`, `bigFancyModuleProduct` etc. If these variables exist only in the scope of `BigFancyModule` package, skip the prefix.
