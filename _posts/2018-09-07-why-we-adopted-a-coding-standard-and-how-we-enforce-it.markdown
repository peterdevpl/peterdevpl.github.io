---
layout: post
title:  "Why we adopted a coding standard and how we enforce it"
date:   2018-09-07 17:00:00 +0100
description: PSR-2 and PSR-4 standards provided great help unifying the PHP coding style in my teams. We enforce them with Continuous Integration and static code analysis.
permalink: /2018/09/07/why-we-adopted-a-coding-standard-and-how-we-enforce-it/
tags:
  - continuous integration
  - php
  - phpcs
  - phpmetrics
  - phpstan
  - psr
  - testing
---

Everyone might have their own code formatting preferences. Problems arise when a team consisting of a couple of individuals work on a common code base and every developer has different preferences. It’s hard to maintain pull requests in this situation.

The most ridiculous and unproductive quarrels in my career were about whether we should use tabs or spaces; should we place a curly bracket in the same line or another; should we leave a blank line at the end of file; etc. It’s mostly a matter of a personal preference, however both sides had some interesting arguments.

Once I told my team that we should adopt common code formatting rules for PHP. It was clear to us that we had a mess in our repos. I asked if we wanted to waste our time endlessly discussing every formatting detail. This way we adopted [PSR-2 standard](http://www.php-fig.org/psr/psr-2/).

Of course we couldn’t just reformat all our repos at once. Every one of seven PHP developers already worked on their branches. We had new things merged to `develop` or `master` a couple of times per day. This is why we were reformatting our code piece by piece. We were choosing the best moments to do a global reformat in PhpStorm. It was easiest when we knew that only one developer at a time was working on a certain repo or module.

It is important to place a code reformatting operation in a separate commit. You shouldn’t mix formatting changes with functional changes because it makes browsing pull requests difficult. Some Git GUIs can hide whitespace changes but they cannot hide syntax changes like `array()` to `[]`.

## Automatic code checks and analysis

To ensure that our coding standards are met, we set up [PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer) in some repos. It verifies that the code complies to the PSR-2 coding standard. After every push to the central repository we get a message about formatting errors. Our testing and analysis tools are automatically launched by the CI (Continuous Integration) tool, for example [GitLab](https://about.gitlab.com/product/continuous-integration/), [Jenkins](https://jenkins.io/), [Travis](https://travis-ci.org/), [CircleCI](https://circleci.com/), [Bamboo](https://www.atlassian.com/software/bamboo).

You should notice that `phpcs` by default enforces that no classes remain in the global namespace. It’s good because, believe me, code can grow really fast and it becomes more and more cumbersome to understand the sophisticated code base without proper namespace structure – best done according to [PSR-4](http://www.php-fig.org/psr/psr-4/).

We can try some other *static code analysis* tools as well: [PHPStan](https://github.com/phpstan/phpstan), [PHPMetrics](https://www.phpmetrics.org/) etc. They do very good job of finding basic and most common code flaws. During the pull requests we can focus on checking advanced business logic because we know that the code formatting and basic code smells were already checked automatically. It’s important especially for dynamically-typed, interpreted languages like PHP where there is no compilation process. Also, PHPStan helps us prepare the code for new PHP versions.
