---
layout: post
title:  "Defensive coding: Final properties and proper autowiring in Spring"
date:   2019-05-31 17:00:00 +0100
description: Getting rid of the "Field injection is not recommended" warning.
permalink: /2019/05/31/defensive-coding-final-properties-and-proper-autowiring-in-spring/
tags:
  - defensive coding
  - java
  - lombok
  - spring
---

### How to get rid of “Field injection is not recommended” warning (and what’s the problem)?

I work in a project where most dependencies in Spring are declared this way:

```java
@Component
class DocumentsController {
    @Autowired
    private UsersRepository users;

    @Autowired
    private DocumentsRepository documents;

    @Autowired
    private PdfGenerator pdfGenerator;
}
```

For every `@Autowired` annotation above, my favorite IDE complains that “field injection is not recommended”. What does it mean?

It’s a nasty hack. I believe every class should be explicit about its dependencies, especially the required ones. They should go into a constructor. Anyone trying to instantiate above class manually should immediately see what is needed.

Dependencies should not be mutable. In the example above, you cannot use `final` for properties because your compiler will complain that the objects are not initialized. The compiler doesn’t know the Spring magic behind instantation of these objects. This means that someone can accidentally overwrite the property.

Let’s refactor that code:

```java
@Component
final class DocumentsController {
    private final UsersRepository users;
    private final DocumentsRepository documents;
    private final PdfGenerator pdfGenerator;

    @Autowired
    public DocumentsController(final UsersRepository users,
                               final DocumentsRepository documents,
                               final PdfGenerator pdfGenerator) {
        this.users = users;
        this.documents = documents;
        this.pdfGenerator = pdfGenerator;
    }
}
```

If you don’t want to write boilerplate code, you can use Lombok’s `@RequiredArgsConstructor` to create the constructor automatically. Still, you benefit from using final and having explicit dependencies (but this requires having a Lombok plugin for your IDE).
