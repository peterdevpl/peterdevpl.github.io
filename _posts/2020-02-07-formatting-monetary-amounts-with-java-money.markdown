---
layout: post
title:  "Formatting monetary amounts with Java Money"
date:   2020-02-07 17:00:00 +0100
permalink: /2020/02/07/formatting-monetary-amounts-with-java-money/
tags:
  - java money
  - java
  - money
---

Java is very often used to develop big financial systems, yet still the JDK has little or no support for monetary arithmetics, currency conversion, formatting money for different locales etc. There’s only a [Currency class](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Currency.html) which serves as a list of ISO 4217 currencies.

Fortunately we’ve got the [JSR-354 standard](https://download.oracle.com/otndocs/jcp/money_currency-1_0-fr-eval-spec/) and [Moneta](https://github.com/JavaMoney/jsr354-ri) – the reference implementation. It provides us the [Martin Fowler’s money pattern](https://www.martinfowler.com/eaaCatalog/money.html) along with string formatting and currency repositories.

In this article I will focus on displaying monetary amounts for different currencies, locales and personal preferences.

## Basic example: amount and ISO symbol

```java
final Locale[] locales = {
    Locale.US,
    new Locale("nl", "NL"),
    new Locale("pl", "PL")
};

final Money amount = Money.of(12345.67, "USD");

for (Locale locale : locales) {
  MonetaryAmountFormat formatter = MonetaryFormats.getAmountFormat(locale);
  System.out.println(formatter.format(amount));
}
```

| en_US        | nl_NL         | pl_PL         |
| ------------ | ------------- | ------------- |
| USD12,345.67 | USD 12.345,67 | 12 345,67 USD |

## Amount with a currency symbol

```java
final Locale[] locales = {
    Locale.US,
    new Locale("nl", "NL"),
    new Locale("pl", "PL")
};

final Money amount = Money.of(12345.67, "USD");

for (Locale locale : locales) {
  MonetaryAmountFormat formatter = MonetaryFormats.getAmountFormat(
      AmountFormatQueryBuilder.of(locale)
          .set(CurrencyStyle.SYMBOL)
          .build());
  System.out.println(formatter.format(amount));
}
```

| en_US      | nl_NL         | pl_PL         |
| ---------- | ------------- | ------------- |
| $12,345.67 | USD 12.345,67 | 12 345,67 USD |

The `$` currency symbol is displayed only for the United States locale. Other languages do not use it.

## Amount with a full currency name

```java
final Locale[] locales = {
    Locale.US,
    new Locale("nl", "NL"),
    new Locale("pl", "PL")
};

final Money amount = Money.of(12345.67, "USD");

for (Locale locale : locales) {
  MonetaryAmountFormat formatter = MonetaryFormats.getAmountFormat(
      AmountFormatQueryBuilder.of(locale)
          .set(CurrencyStyle.NAME)
          .build());
  System.out.println(formatter.format(amount));
}
```

| en_US              | nl_NL               | pl_PL               |
| ------------------ | ------------------- | ------------------- |
| US Dollar12,345.67 | US Dollar 12.345,67 | 12 345,67 US Dollar |

## Amount without a thousands separator

```java
final Locale[] locales = {
    Locale.US,
    new Locale("nl", "NL"),
    new Locale("pl", "PL")
};

final Money amount = Money.of(12345.67, "USD");

for (Locale locale : locales) {
  MonetaryAmountFormat formatter = MonetaryFormats.getAmountFormat(
      AmountFormatQueryBuilder.of(locale)
          .set(AmountFormatParams.GROUPING_SIZES, new int[]{2, 0})
          .build());
  System.out.println(formatter.format(amount));
}
```

| en_US       | nl_NL        | pl_PL        |
| ----------- | ------------ | ------------ |
| USD12345.67 | USD 12345,67 | 12345,67 USD |

Grouping sizes are specified starting from the decimal point. In the above example we have two digits after the decimal point, and then we have no more grouping going to the left. Of course other combinations are possible, e.g. `{2, 3, 4}`.

## Custom pattern

Using a pattern as defined by [java.text.DecimalFormat](https://docs.oracle.com/javase/7/docs/api/java/text/DecimalFormat.html).

```java
final Locale[] locales = {
    Locale.US,
    new Locale("nl", "NL"),
    new Locale("pl", "PL")
};
final Money amount = Money.of(12345.67, "USD");
for (Locale locale : locales) {
  MonetaryAmountFormat formatter = MonetaryFormats.getAmountFormat(
      AmountFormatQueryBuilder.of(locale)
          .set(AmountFormatParams.PATTERN, "###,###.## ¤")
          .build());
  System.out.println(formatter.format(amount));
}
```

| en_US         | nl_NL         | pl_PL         |
| ------------- | ------------- | ------------- |
| 12,345.67 USD | 12.345,67 USD | 12 345,67 USD |
