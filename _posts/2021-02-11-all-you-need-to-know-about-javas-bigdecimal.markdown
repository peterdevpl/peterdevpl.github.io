---
layout: post
title:  "All you need to know about Java’s BigDecimal"
date:   2021-02-11 17:00:00 +0100
permalink: /2021/02/11/all-you-need-to-know-about-javas-bigdecimal/
description: A guide to Java BigDecimal class. Examples of monetary calculations and formatting decimal numbers for different languages.
tags:
  - java
  - money
---

Popular programming languages do not natively support decimal numbers. This is because CPUs operate on binary numbers. Even though there is a new IEEE standard for decimal floating point types, CPUs still don’t support it fully. So every time we see a notation like `0.1` in the code, it’s not what it seems. **Our calculations might be inaccurate.**

Most modern languages have dedicated libraries to handle decimals. Internally, they use either a long integer type or a string to store the number. They implement their own arithmetic engines. In Java, there is a `BigDecimal` class.

**The safest way to create a new number is to use a string as an input:**

```java
final BigDecimal number = new BigDecimal("123.45");
```

> To save memory, special `BigDecimal` instances already exist: `BigDecimal.ZERO`, `BigDecimal.ONE` and `BigDecimal.TEN`. You should reuse them instead of creating your own.

**It is not recommended to use the `double` type** when creating a `BigDecimal` object. Even if we enter a value like `0.1`, the actual representation equals to something around 0.10000000000000000555 which definitely does not look like a monetary amount or anything else that we would expect. This is because `double` is a base-2 scientific notation type. Try running this code to see it for yourself:

```java
System.out.println(0.20 + 0.10);
```

The `BigDecimal` class offers several methods for basic operations like addition, subtraction, multiplication and division. Before we go into calculations, we need to talk more about the internals.

## Precision and scale

`BigDecimal` uses two parameters to define the maximum number of digits it can hold and how many digits are behind the decimal point. The first one is called *precision*, and the other one is called *scale*.

It is very important that you understand what happens to these parameters because they affect rounding and the string representation.

If you use the simplest string constructor like in the examples above, *precision* is set to 0 (which means infinite length) and *scale* is set to the number of digits behind the decimal point. For `123.45`, scale will be 2.

You can use the `setScale()` method to increase scale if you want to show the exact number of digits in a fraction, even if these will be zeros. The number `123.45` with a scale of 4 would be represented as `123.4500`.

Scale can change when you add, subtract, multiply or divide numbers with fractions. This matters especially when you try to calculate taxes. For example, multiplying `123.45` times `1.23` gives us `151.8435`, but this is not a proper monetary amount. You have to perform rounding using the second argument for `setScale()`:

```java
final BigDecimal net = new BigDecimal("123.45");
final BigDecimal tax = new BigDecimal("1.23");
final BigDecimal gross = net.multiply(tax).setScale(2, RoundingMode.HALF_UP);
System.out.println(gross);
// output is 151.84
```

Some numbers do not have a finite decimal representation, like 1/3. They cannot be stored as `BigDecimal` and rounding has to be applied. It’s your responsibility to specify target precision or scale, otherwise division might cause an exception:

```java
final BigDecimal result = BigDecimal.ONE.divide(
    new BigDecimal("3"), 5, RoundingMode.HALF_EVEN);
System.out.println(result);
// output is 0.33333
```

Another operation that involves changing scale is removing trailing zeros. Sometimes, for example after several subtractions, you don’t want to leave zeros at the end. The `stripTrailingZeros()` method will return the same number without trailing zeros.

## Rounding modes

In the previous example you’ve seen an example of rounding. The most popular option is called `HALF_UP` and it is commonly taught at school. You round up when the discarded fraction is greater than or equal to `0.5`; you round down when the fraction is below `0.5`. So for example, assuming a target scale of 2, the number `1.234` will be rounded to `1.23`, and the number `1.235` will be rounded to `1.24`.

However, different taxation laws might require different rounding modes, for example always rounding up. They are listed in an enum called `RoundingMode`.

Below are some trivias which can help you distinguish between different modes:

* `UP` never decreases the magnitude of the calculated value.
* `DOWN` never increases the magnitute of the calculated value.
* `HALF_UP` is commonly taught at school.
* `FLOOR` never increases the calculated value.
* `CEILING` never decreases the calculated value.
* `HALF_EVEN` is also known as “banker’s rounding” because it reduces error when performing multiple operations on rounded numbers. If the first digit outside scale is 5, we round to the nearest even number. Otherwise, standard rules apply.
* `UNNECESSARY` is used to check if rounding was performed or not; if rounding would be necessary, an `ArithmeticException` is thrown.

> Always consult the rounding mode and other assumptions with an accounting or taxation expert. It is their responsibility to make decisions according to the law, and your responsibility is only to write reliable software that implements these rules.

## Understanding MathContext

The `BigDecimal` class uses rules defined by a `MathContext` to perform numerical operations. In most cases you won’t need to worry about it. However, we should get back to the example of dividing 1 by 3.

By default, `BigDecimal` numbers have “unlimited” precision. In fact, the maximum unscaled value is equal to 2^Integer.MAX_VALUE, according to the `BigInteger` documentation. This looks like more than enough to represent any finite number you need.

Nevertheless, we don’t want to run out of memory when doing a simple division of 1 by 3. Earlier, we just specified a desired scale and a rounding mode, but you should be also aware that you can control precision of such operation.

There are three `MathContext` objects that correspond to the IEEE 754R decimal formats. `DECIMAL32`, `DECIMAL64` and `DECIMAL128` allow a maximum number of 7, 16 and 34 digits, respectively. They all use the `HALF_EVEN` rounding mode. You can use these contexts to control division:

```java
final BigDecimal result = BigDecimal.ONE.divide(
    new BigDecimal("3"), MathContext.DECIMAL32);
System.out.println(result);
// output is 0.3333333
```

## Immutability

A very important concept of Java `BigDecimal` type is immutability. It means that once an object is instantiated, its state cannot be changed. The only way to obtain a modified object is to create a new instance:

```java
final BigDecimal number1 = new BigDecimal("99");
number1.add(BigDecimal.ONE);
System.out.println(number1);
// number1 is still 99
```

This behavior prevents many bugs that could occur if we passed an object to other methods and they unexpectedly altered the object’s state.

## String representation

A standard way to output a `BigDecimal` object on the screen is to just use the `toString()` method. There are two other methods though, and it’s worth to know them.

The difference is visible when we operate on numbers written using scientific notation, like `1.23E+3`, which is equal to `1230`. The `toString()` method will create a string in that notation, while `toPlainString()` will always return the full number. `toEngineeringString()` is a variation where the exponent is always a multiple of three (if an exponent is needed at all).

| Input number | toString() | toEngineeringString() | toPlainString() |
| ------------ | ---------- | --------------------- | --------------- |
| 1.23E2       | 123        | 123                   | 123             |
| 1.23E3       | 1.23E+3    | 1.23E+3               | 1230            |
| 1.23E4       | 1.23E+4    | 12.3E+3               | 12300           |

Just to remind, you can use `stripTrailingZeros()` to strip unnecessary zeros from fractions:

```java
final BigDecimal numberWithZeros = new BigDecimal("1.000");
System.out.println(numberWithZeros);
// output is 1.000

final BigDecimal strippedNumber = numberWithZeros.stripTrailingZeros();
System.out.println(strippedNumber);
// output is 1
```

The only problem with all the examples above is that they don’t conform to language rules other than English. What if we want to make an international application?

### Using locale for number formatting

Most programming languages assume English notation for numbers. They use a dot to separate decimal part from an integer part. When presenting a number to a user, we can optionally separate thousands with comma.

However, many languages and countries have different regulations. If our application is dedicated for international markets, localization is a very important matter we should take into account.

To make localization easier, a concept of a locale was introduced. A locale is a “set of parameters that defines the user’s language, region and any special variant preferences that the user wants to see in their user interface.” ([Wikipedia](https://en.wikipedia.org/wiki/Locale_(computer_software)))

A locale identifier combines language and country code. So for British English we have `en_GB`, American English is `en_US`, and Swiss German will be `de_CH`.

Let’s analyze how a sample number would be formatted using some of the world’s locales. We’ll pick *twelve thousand three hundred forty five point sixty seven*, which can be written as `12345.67` in the code:

| Language | Country       | Locale code | Formatted value |
| -------- | ------------- | ----------- | --------------- |
| English  | United States | en_US       | 12,345.67       |
| Polish   | Poland        | pl_PL       | 12 345,67       |
| Spanish  | Spain         | es_ES       | 12.345,67       |
| Spanish  | Mexico        | es_MX       | 12,345.67       |

Notice the difference for Spanish language. In Spain, people use a dot to separate thousands and a comma as a decimal separator. In Mexico, it’s the other way around, just like in the U.S. It means that it’s not enough to localize your application for a specific language; the region is important too.

### Formatting and parsing numbers with NumberFormat

An abstract class called `NumberFormat` has multiple `getInstance()`-like methods that we can use to create a localized number format, depending on our needs. As the only argument, we should specify a desired locale. If we skip this, the default system locale will be used.

```java
final BigDecimal result = new BigDecimal("12345.67");
final NumberFormat numberFormat = NumberFormat.getInstance(Locale.forLanguageTag("en_US"));
System.out.println(numberFormat.format(result));
// output is 12,345.67
```

The number format can be further customized. For example, you can turn grouping off by calling `numberFormat.setGroupingUsed(false)`.

You can also use `NumberFormat.getPercentInstance()` to create a percentage format. This way, a number like `0.51` will be presented as `51%`. Such format is useful to print a tax rate.

Here’s an extended version of the code to calculate tax and gross values – typical data on every sales invoice:

```java
final BigDecimal net = new BigDecimal("123.45");
final BigDecimal taxRate = new BigDecimal("0.23");
final BigDecimal tax = net.multiply(taxRate).setScale(2, RoundingMode.HALF_UP);
final BigDecimal gross = net.add(tax);

final NumberFormat numberFormat = NumberFormat.getCurrencyInstance(Locale.forLanguageTag("en_US"));
numberFormat.setCurrency(Currency.getInstance("USD"));
final NumberFormat percentFormat = NumberFormat.getPercentInstance(Locale.forLanguageTag("en_US"));

System.out.println("Net value:   " + numberFormat.format(net));
System.out.println("Tax value:   " + numberFormat.format(tax));
System.out.println("Tax rate:    " + percentFormat.format(taxRate));
System.out.println("Gross value: " + numberFormat.format(gross));

/* output is:
Net value:   USD 123.45
Tax value:   USD 28.39
Tax rate:    23%
Gross value: USD 151.84
 */
```

## Wrapping up

Decimal calculations need extra care. Computers do not support decimal numbers natively, so we have to use dedicated libraries like `BigDecimal`.

Accuracy is especially important for monetary calculations. I recommend using the [Java Money library](https://javamoney.github.io/) as it also introduces handling currencies. However, knowing the `BigDecimal` class can still be useful.

*This is a part of the book I’m writing – “Money explained to Java developers”. If you’re interested, [follow me on Twitter](https://twitter.com/peterdevpl) for more insights!*