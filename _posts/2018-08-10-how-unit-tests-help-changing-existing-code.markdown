---
layout: post
title:  "How unit tests help changing existing code"
date:   2018-08-10 17:00:00 +0100
description: A short history about PHP unit tests which allowed me to easily change an unknown business logic code.
permalink: /2018/08/10/how-unit-tests-help-changing-existing-code/
tags:
  - php
  - phpunit
  - testing
---

You should have some tests. Why? Because **developers are afraid to make changes in a code they don’t understand.** This is a common problem not only with fresh employees, but with everyone who stands in front of a huge, complex, legacy system. They are afraid to add a new if not to break the other ones. They are afraid to erase code that seems obsolete.

Instead of refactoring, people tend to add classes and methods with suffixes like `_new`, `_2` and so on. I saw the same fear of changes in QA and op teams. When I try to refactor things, I often hear: *Oh, maybe not now in case if something breaks, maybe later…*

## Overcoming the fear of changing an unknown code

I had a problem changing a piece of complex code adding symbols to online transactions. The symbols depended on product types, legal issues, invoices etc. The sales dep coined [some weird terms which developers didn’t understand](https://martinfowler.com/bliki/UbiquitousLanguage.html), and those terms were very important for them. I was told to add yet another weird symbol on top of that. The original code looked like this:

```php
public function getDescription()
{
    if (\in_array($this->getItemsType(), [self::ORDER_MATERIAL, self::ORDER_MIXED], true)) {
        $type = 'DW';
    } elseif (self::ORDER_COURSE === $this->getItemsType()) {
        $type = 'SzO';
    } else {
        $type = $this->hasInvoice() ? 'M' : 'P';
    }

    return sprintf('Order #%u / %s', $this->getId(), $type);
}
```

After analyzing the original code, I wrote a unit test for all cases:

```php
use PHPUnit\Framework\TestCase;
use Piotr\Blog\Entity\Order;

final class OrderTest extends TestCase
{
    /**
     * Test data will be provided by ordersProvider().
     * @dataProvider ordersProvider
     */
    public function testDescription(int $orderId, int $itemsType, bool $hasInvoice, string $expected)
    {
        $order = new Order($orderId);
        $order->setItemsType($itemsType)->setHasInvoice($hasInvoice);
        $this->assertEquals($expected, $order->getDescription());
    }

    public function ordersProvider()
    {
        return [
            [123, Order::ORDER_MULTIMEDIA, false, 'Order #123 / P'],
            [123, Order::ORDER_MATERIAL, false, 'Order #123 / DW'],
            [123, Order::ORDER_MIXED, false, 'Order #123 / DW'],
            [123, Order::ORDER_COURSE, false, 'Order #123 / SzO'],
            [123, Order::ORDER_MULTIMEDIA, true, 'Order #123 / M'],
            [123, Order::ORDER_MATERIAL, true, 'Order #123 / DW'],
            [123, Order::ORDER_MIXED, true, 'Order #123 / DW'],
            [123, Order::ORDER_COURSE, true, 'Order #123 / SzO'],
        ];
    }
}
```

I checked [code coverage](https://phpunit.readthedocs.io/en/7.4/code-coverage-analysis.html) for this method, by curiosity. It was 100% which made me confident that all lines are executed during the test (however, it might not ensure that all cases are checked). Now I was ready to add another condition to the `getDescription()` method:

```php
public function getDescription()
{
    if (self::ORDER_VIDEO === $this->getItemsType) {
        $type = $this->hasInvoice() ? 'W' : 'WP';
    }
    /* ... */
}
```

I ran my unit test and received no errors. Success – I didn’t break anything! Since I added some extra lines to my class, the code coverage dropped. I needed to add new test cases:

```php
public function ordersProvider()
{
    return [
        /* ... */
        [123, Order::ORDER_VIDEO, false, 'Order #123 / WP'],
        [123, Order::ORDER_VIDEO, true, 'Order #123 / W'],
    ];
}
```

Test is passing and my code coverage is 100% again. Now I know that another developer who takes this code over will have a trustful test checking all conditions.

## Make sure your business works!

The above example is easy. Unfortunately, in every day job writing unit tests is hard if we need to deal with a poor system architecture, [tight coupling](http://fabien.potencier.org/what-is-dependency-injection.html) and… managers refusing the team to spend extra time writing tests. Moreover, unit tests alone might not be sufficient – you would want integration and UI tests also, and it takes time. [But, as Robert C. Martin ironically pointed out once:](https://youtu.be/o_TH-Y78tt4?t=1833)

> Can you imagine telling your users: You know, I don’t write tests. I just write the code. Sometimes it even works. And I ship it to you, and if there are bugs, you’re going to tell me, aren’t you?

**Your company might not need a 100% code coverage** – often it’s very difficult and even unprofitable ([some people claim it’s dangerous](https://labs.ig.com/code-coverage-100-percent-tragedy)). From the business perspective, we should write tests that protect the key business issues first – and that’s usually the domain logic. We want to make sure that none of the business rules that we agreed upon will break after deploying new features. We want to make sure that users will still be able do place orders, and all the orders will be properly accounted.

However if you want to write tests for business rules, then those rules have to be clearly written in the code. This is often cumbersome – and I’m going to tell you about it, some day.
