---
layout: post
title:  "When your SQL database is missing foreign keys"
date:   2018-10-19 17:00:00 +0100
description: A guide to introducing foreign key constraints in an existing SQL database. You will learn how to fix erroneous data which does not apply to the constraints.
permalink: /2018/10/19/when-your-sql-database-is-missing-foreign-keys/
tags:
  - database
  - sql
---

…then sooner or later, you’re going to have a bad time. Bugs in your app or users’ recklessness will cause your database to be inconsistent.

An example from my job: a system had `users` and `users_categories` tables. While registering their accounts, new users entered not only an e-mail address and a password, but also selected categories, like *teacher*, *student*, *parent*. The data were immediately inserted into a MySQL database. But the account had to be activated via e-mail.

A script was executed every day to purge inactive accounts. It wiped records only from the `users` table, not `users_categories`. There were no foreign keys to block that behavior.

Every company has some erroneous legacy code here and there. I saw databases reaching 60-70 gigabytes in size, with hundreds of millions of records and not having foreign key constraints because someone… was afraid of them. A long time ago, one database in the company had `ON DELETE CASCADE` foreign keys, which means deleting one record caused cascaded deletions of related records. My colleague destroyed almost half of a test database, so he decided to remove foreign key constraints. That was a classic misunderstanding of the technology that we were using.

## How a foreign key constraint works?

In a relational database, foreign keys are meant to secure the references between entities. For example, in a `users_categories` we have two relations: with `users` and `categories` tables. No row in the `users_categories` table cannot reference a non-existent user or category.

The only exception is if a row contains `NULL` values. This is a way to create optional relations.

Creating a foreign key usually looks like that (based on InnoDB engine in MySQL):

```sql
ALTER TABLE users_categories
    ADD CONSTRAINT fk_users_categories_user_id
        FOREIGN KEY (user_id) REFERENCES users (id)
        ON DELETE RESTRICT
```

We create a constraint with a specified name (we don’t have to, but it’s good to have a name that we can later use during reverting a migration). We specify a column which should be restricted and a destination table and column which we want to refer to. In the end, we decide what happens if we try to delete related data. A default behavior is to restrict such query and issue an error, for example if we try to remove a user without removing their categories list. We can also decide to have a cascade deletion or set NULLs, but I always choose the safest option which is to `RESTRICT`.

### Foreign keys require that:

* both columns (the one we create a constraint for and the one we refer to) must have exactly the same type; a `SMALLINT` column cannot refer to `INT`,
* there must be an index in the source table for the column; it can be a multi-column index starting with that column; if no index is matching, a new one will be created
* a constraint must have a unique name in the schema scope

## Introducing foreign keys in an inconsistent database

If you have some existing data without foreign keys, you need to clean it up first. That’s a challenge because sooner or later, a database without foreign keys will be a mess.

MySQL will not allow us to create a foreign key on wrong data – unless we issue a `SET FOREIGN_KEY_CHECKS=0` query before adding constraints (that’s what `mysqldump` does by default). However, we would like to have proper data not only until now, but also to fix what we already have. We need to somehow untangle that existing spaghetti.

**I decided to analyze existing data** to know:

* how many records with wrong references do I have
* what strategy will be the best to fix these particular records
* are there differences in column types
* On every table I executed a query showing all records with wrong references:

```sql
SELECT * FROM users_categories WHERE NOT EXISTS (
    SELECT * FROM users WHERE user_id = users.id
)
```

In the example above, if the `user_id` column would allow NULL values, we should add a following condition: `user_id IS NOT NULL`.

I noticed a funny trend between developers not using foreign keys: they define optional references as, for example, `INT NOT NULL DEFAULT 0`. This is not correct because usually, in referenced tables with `AUTO_INCREMENT` or `SERIAL` primary keys, there are no records with `id = 0`. So introducing a foreign key in this case will not work. I had to modify the table schema and then change all 0s to NULLs. Let’s say that a user might have an optional reference to a city:

```sql
ALTER TABLE users MODIFY city_id INT DEFAULT NULL;

UPDATE users SET city_id = NULL WHERE NOT EXISTS
  (SELECT * FROM cities WHERE city_id = cities.id);
```

Here we can see a strategy of setting `NULL` every time we cannot find a destination entity. We don’t want to remove a user’s record just because it refers to a non-existent city. `NULL` means a value is unknown, uncertain (it’s not the same as an empty value or 0). If we have an incorrect `city_id` value, we cannot specify which exact city it refers to – so we set it to `NULL`.

Other strategy can be taken for many-to-many join tables. In the `users_categories` table I mentioned, if we have invalid `user_id`, `category_id` or even both – this means that the whole record can be removed:

```sql
DELETE uc FROM users_categories uc WHERE NOT EXISTS (
    SELECT * FROM users u WHERE u.id = uc.user_id
) OR NOT EXISTS (
    SELECT * FROM categories c WHERE c.id = uc.category_id
)
```

When you pick a certain data cleanup strategy, you need to know the business well. Maybe the wrong data is used in some reporting systems and if you suddenly fix the stats, it will confuse people because they relied on these data for a long time. They might perceive your ingenious fix… as a mistake 🙂

## Fixing complex cases

Sometimes, relations between entities form a long chain: for example, a user puts and order which consists of several items, and every order item refers to a product, and every product… has been added by some admin user. At first you should check the most generic entities (like products) and then dig deeper into orders and order items. Take a look at the following steps to see which strategy I pick in different scenarios:

1. Check if there are products added by non-existent users. An information about a removed user is not necessary for the system to work. A wrong `user_id` can be set to `NULL`.
2. Check if there are orders with wrong `user_id`. An orders history is essential for the legal, tax and reporting purposes. We cannot remove any orders, so we just set `user_id = NULL` if a user does not exist (maybe he was not logged in?).
3. Check if there are any order items related to non-existent orders or products. Items contain important invoicing data like net and gross price. But if a report requires a proper reference from `orders_items` to `orders` table, and we have queries like `SELECT … FROM orders JOIN orders_items …`, then a `JOIN` clause eliminates wrong records anyway. So we can remove items that refer to wrong orders. The same situation applies to `product_id`. We won’t prepare sale reports for products which do not exist.

Of course before executing such dangerous queries we need to have a backup. And while we analyze the details of the system we’re trying to clean up, we should consult as many people who have a business knowledge as possible.

I know it would be nice to have an automatic script to clean up all the data before adding constraints. But there is no silver bullet because every case is different. You need to use your creativity and intuition! It’s not easy, but in the end, satisfaction will be great.

### Some interesting, further, external read:

* [9 Reasons Why There Are No Foreign Keys In Your Database](https://dataedo.com/blog/why-there-are-no-foreign-keys-in-your-database-referential-integrity-checks)
* [Major Applications Do Not Use Foreign Key Constraints In Their Databases](https://dataedo.com/blog/major-applications-do-not-use-foreign-key-constraints-in-their-databases-oracle-microsoft-sap)
