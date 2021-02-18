---
layout: post
title:  "How legacy code is made – part 2"
date:   2018-07-27 17:00:00 +0100
permalink: /2018/07/27/how-legacy-code-is-made-part-2/
tags:
  - legacy
  - php
  - technical debt
---

[In the previous post](/2018/07/13/how-a-legacy-code-is-made/), I told you how tight coupling causes systems to be hard to maintain and expand with new features. Today I’m going to show you how developers often take shortcuts which then add up to the technical debt.

I worked on a big e-commerce system for one of the leading Polish education publishers. When I joined the team, there were already many crazy workarounds in the system. I found one particular gem which was really funny and perfectly showcased the absurd of taking ridiculous shortcuts. Let me describe it.

There was a feature for wholesale customers to import a [CSV file](https://en.wikipedia.org/wiki/Comma-separated_values) with all the products they wanted to order. The import script read the CSV line by line and placed every product into the order. Every line started with a product code consisting of several letters and numbers. The Polish alphabet, apart from standard Latin letters, contains 9 letters with [diacritics](https://en.wikipedia.org/wiki/Diacritic#Languages_with_letters_containing_diacritics) (accents). They are not contained within the basic ASCII set, so at some point the team ran into character encoding issues.

This ticket was found in the issue tracker:

> Issue title: Cannot import CSV file with diacritics
>
> Issue description: The import function do not recognize RŁ and RŁ2 product codes. Under Windows, CSV files have a default CP-1250 encoding. The database uses UTF-8 encoding. For these two product codes, we need to add special conditions.

Notice that the last sentence of the issue description is a trap; it suggests creating a *workaround*, some special case in the code. One PHP developer fell into that trap and implemented a fix like this:

```php
while (($csv = fgetcsv($fp, 256, $delimiter)) !== false) {
    // for CP-1250 files we convert Ł letter to UTF-8
    $csv[0] = str_replace("\xA3", "\xC5\x81", $csv[0]);
    // ...
}
```

Now imagine that we have some product codes with another diacritic letters. What would an average developer do? Add another special case for converting that character? It is really tempting to do just that, then commit the change and forget about it.

**This is the problem with workarounds:** people easily follow them instead of solving the problem properly. Not everyone has the courage to modify the code they did not write. People think, “If it’s been around here for several months or years, it must be correct. Someone who did this must have known what he or she was doing.”

In this specific example we could simply use PHP’s [iconv](http://php.net/manual/en/function.iconv.php) function to solve the encoding issue properly. So it’s hard to say that anyone has actually made a *shortcut* here; in fact, the original solution was a much longer walk than it should be.

## Don’t rush! You won’t save time!

We all do bad things when working under pressure of deadlines. But often a hurry makes us actually work slower. Trying to take a shortcut and creating workarounds has its consequences which then are stacked to something we call *technical debt*. Think, research and ask before you code.

New developers who join the team do not know the project and its practices. They just follow examples in the existing code base. This can lead to funny and awkward situations. We should spend more time on leaving good examples for our future coworkers (and ourselves). If we can’t make a good code before the deadline, we should clearly mark our workarounds and schedule some refactoring as soon as possible.
