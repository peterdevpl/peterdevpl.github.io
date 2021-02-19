---
layout: post
title:  "Sending monetary amounts over network"
date:   2020-11-22 17:00:00 +0100
permalink: /2020/11/22/sending-monetary-amounts-over-network/
description: Quick guide on avoiding common problems when sending monetary amounts through an API, SQL or CSV. How to handle money without bugs?
tags:
  - java money
  - money
  - moneyphp
---

If you write any application that operates on money – even a simple web shop, sooner or later you will have to send monetary amounts through API calls, SQL queries, CSV downloads, and so on. Let’s see how to avoid common mistakes which could destroy your business!

## Money over an API

Whether you request a payment via an external API, or expose your own data to your clients, it is important to set up a protocol. The most popular data format for API calls is JSON. However, this format accepts mostly anything you’ll feed it with, so **it’s not going to magically guess that you’re sending financial data** – like the product info below:

```json
{
  "id": 1,
  "name": "Keyboard",
  "price": 22.99,
  "availability": "in stock"
}
```

What does `22.99` mean here? What is the currency? Are you aware that you’re using a float data type here, which makes the amount imprecise? And if there was an integer like 23 – is it an amount of dollars or cents?

The above piece of JSON would look a lot better like this:

```json
{
  "id": 1,
  "name": "Keyboard",
  "price": {
    "amountDecimal": "22.99",
    "currency": "USD"
  },
  "availability": "in stock"
}
```

Of course instead of `amountDecimal` you can use `amountInteger` if you prefer. The key is to let everyone know what exactly that value means!

## Money in a SQL database

Unfortunately, SQL databases aren’t helpful with storing monetary data, so you have to take care of it by yourself.

There are two solutions:

1. Use one `BIGINT` or `DECIMAL/NUMBER` column to store the amount, and a `CHAR(3)` column to store currency code. This way you’ll be able to sort by price or do some calculations directly in SQL queries, however be careful not to mix different currencies.
2. Use one `VARCHAR` column to encode the whole value, like `USD 123` or `USD 1.23` (depends if you prefer using cents or dollars).

Never use the `FLOAT` type which is not suited for precise decimal calculations!

## Money in a CSV file

This one can be done horribly wrong without proper encoding. CSV stands for Comma-Separated Values. An example CSV file containing a list of products can look like this:

```
ID,Name,Price,Availability
1,Keyboard,22.99,in stock
2,Mouse,5.99,out of stock
```

**The comma is a special character** here which is used to separate columns. However in some languages, comma acts as a decimal separator, so you would have `22,99` instead of `22.99`. This would break the table while parsing!

You can solve this issue by using a **dedicated CSV library** for data export. Any values containing a comma will be automatically encoded. Usually the whole value is enclosed with quotes or the comma inside a value is doubled.

Two more questions arise:

1. **What is the currency in the file above?** Do these numbers refer to dollars, euros, or something else? There should be a separate column specifying the currency!
2. While parsing, how do we know **what number format was used?** Should we expect a dot or a comma inside a number? What language rules should we use?

## Take care of your data!

When exchanging financial data with other systems, always make sure to:

1. **Use a proper unit:** either dollars or cents, either euros or cents, and so on.
2. **Never use binary floating-point numbers.** Send a string like `"19.99"`, not just `19.99`. Or use integers.
3. **Specify currency** using the ISO 4217 standard.
4. **Respect the encoding rules** of the data format you’re using.
5. **Parse localized money strings** properly.
