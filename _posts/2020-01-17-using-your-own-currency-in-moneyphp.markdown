---
layout: post
title:  "Using your own currency in MoneyPHP"
date:   2020-01-17 17:00:00 +0100
permalink: /2020/01/17/using-your-own-currency-in-moneyphp/
description: Create custom (non-ISO) currencies with MoneyPHP, convert them and format output.
tags:
  - php
  - money
  - moneyphp
---

I have this [talk about the MoneyPHP library](https://www.slideshare.net/PiotrHorzycki/how-to-count-money-using-php-and-not-lose-money) and handling currencies in general. I often get a question: is it possible to define custom currencies in [MoneyPHP](https://moneyphp.org/en/stable/index.html)? A possible use case would be loyalty cards for example, with their own scoring system.

The answer is: YES! MoneyPHP not only accepts all [ISO 4217 currencies](https://www.iso.org/iso-4217-currency-codes.html), but also allows you to define a custom list with any identifiers and any subunits. So for example, you can create a currency called `BlahBlahBlah` that has 7 decimal points and then convert an amount from your currency to another by providing a custom exchange list.

In the example below, we’re creating two fictious currencies: `MyPoints` and `TheirPoints`. The first one has two decimal points while another has none (only whole numbers). We create a `Money` object that holds exactly 123.45 MyPoints.

Then we specify an exchange table which says that 1 TheirPoint equals 0.5 MyPoints. We create a currency converter and then try to convert one currency to another. `123.45 * 0.5` equals `61.725`. But since TheirPoints subunit is 0, the amount has to be rounded up, so we get 62.

```php
use Money\Converter;
use Money\Currencies\CurrencyList;
use Money\Currency;
use Money\Exchange\FixedExchange;
use Money\Money;

$currencyList = new CurrencyList([
    'MyPoints' => 2,
    'TheirPoints' => 0,
]);

$myCardAmount = new Money(12345, new Currency('MyPoints'));

$exchange = new FixedExchange([
    'MyPoints' => [
        'TheirPoints' => 0.5,
    ],
]);
$converter = new Converter($currencyList, $exchange);

$theirCardAmount = $converter->convert(
    $myCardAmount, new Currency('TheirPoints')
);

echo $theirCardAmount->getAmount();  // 62
```

However, there can be a problem with currency formatting using PHP’s `intl` library. Consider this example:

```php
$numberFormatter = new \NumberFormatter(
    'en_US', \NumberFormatter::CURRENCY
);
$moneyFormatter = new \Money\Formatter\IntlMoneyFormatter(
    $numberFormatter, $currencyList
);

echo $moneyFormatter->format($theirCardAmount);  // The 62.00
```

The odd output (`The` instead of `Their` and the unwanted fractional part) comes from the fact that [PHP’s NumberFormatter](https://www.php.net/manual/en/numberformatter.formatcurrency.php) handles only ISO currencies which have 3-character symbols. So you should use `\NumberFormatter::DECIMAL` instead and add your custom symbol manualy.

[See my post about handling money in PHP](/2018/08/24/learn-how-to-count-money-or-you-will-lose-it/)
