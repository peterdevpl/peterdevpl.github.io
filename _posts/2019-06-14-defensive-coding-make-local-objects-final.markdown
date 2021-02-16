---
layout: post
title:  "Defensive coding: Make local objects final"
date:   2019-06-14 17:00:00 +0100
permalink: /2019/06/14/defensive-coding-make-local-objects-final/
tags:
  - defensive coding
  - java
  - javascript
  - php
---

In the [very first article on defensive coding](/2019/04/19/defensive-coding-avoiding-mutability-and-side-effects/) we talked about avoiding mutability.

Let’s talk about JavaScript for a while. Every modern tutorial says that you can instantiate new objects either with `let` or `const`. You use `let` if you intend to create a variable (like with an obsolete `var` keyword), or you can use const to indicate that an object – once initialized – should remain constant. This gives developers great clarity and saves them from mistaken mutations.

What about Java? You don’t have a `const` keyword, but you have `final`. I love using it and many developers in my current project also use `final` wherever feasible. You can mark properties, local objects and method arguments as `final`. There are very rare cases when we need to mutate an object, so must of the time I end up having final everywhere:

```java
public BigDecimal getTotalPrice(final Product product,
                                final BigInteger quantity) {
    final BigDecimal totalNetPrice = product
             .getUnitPrice()
             .multiply(quantity);

    return totalNetPrice.multiply(product.getTaxRate());
}
```

Why use `final` everywhere? It can look strange in the beginning, but it protects us from bugs. How many times have you seen robust methods spanning multiple screens, with vague and similar variable names? How many times have you mistaken `foo`, `oldFoo`, `databaseFoo`, `foo1` etc.?

In PHP, the `final` keyword can be applied only to classes. You can’t create local constants, only class constants with `const` keyword. It’s even worse that you don’t clearly distingish first object instantiation from subsequent instantations using the same variable name. This code is perfectly valid in PHP:

```php
$builder = new ProductBuilder();
$builder = new UserBuilder();
```
