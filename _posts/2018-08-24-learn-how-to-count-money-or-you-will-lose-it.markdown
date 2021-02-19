---
layout: post
title:  "Learn how to count money… or you will lose it"
date:   2018-08-24 17:00:00 +0100
description: An example on how floating point numbers are stored using IEEE-754 and what bugs can you encounter. How to store monetary values using PHP and MoneyPHP?
excerpt: "It sounds weird when I get a ticket like this: when I set the price to $4.20 everything’s fine; but I cannot set the price to $4.10 because the system shows $4.09. I did some research and discovered that a user entered the price in dollars and then we converted it to cents to store in the database as an integer. That’s where the mistake was made."
permalink: /2018/08/24/learn-how-to-count-money-or-you-will-lose-it/
tags:
  - money
  - php
---

Does anyone know where does the following difference come from?

```
$ php -r "var_dump((int) (4.20 * 100));"
int(420)

$ php -r "var_dump((int) (4.10 * 100));"
int(409)
```

It sounds weird when I get a ticket like this: *When I set the price to $4.20 everything’s fine. But I cannot set the price to $4.10 because the system shows $4.09*. I did some research and discovered that a user entered the price in dollars and then we converted it to cents to store in the database as an integer. That’s where the mistake was made.

Those problems arise from the way that CPU stores floating point numbers. PHP does not have a built-in decimal type, so it uses IEEE-754 standard. In this standard, numbers are stored in a binary system; both the mantissa and the exponent.

You can play with converting different numbers here: [IEEE-754 Floating Point Converter](https://www.h-schmidt.net/FloatConverter/IEEE754.html). You can see that the number 4.10 in fact is stored as 4.099999904632568359375. When you multiply it by 100, you get around 409.99999. Casting to `(int)` causes dropping the fractional part (not rounding), so we get the number 409.

How to operate with currencies in PHP? The best way is to use [decimal types](http://php.net/manual/en/ref.bc.php) or [dedicated currency classes](http://moneyphp.org/en/latest/). `Money` is a classic example of a value object as a Domain-Driven Design building block.

> If I had a dime for every time I’ve seen someone use FLOAT to store currency, I’d have $999.997634 – Bill Karwin
