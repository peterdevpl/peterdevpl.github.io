---
layout: post
title:  "Testing PDF documents"
date:   2018-12-28 17:00:00 +0100
description: An example of parsing PDF document contents and testing it with Java, JUnit and PDFBox.
permalink: /2018/12/28/testing-pdf-documents/
tags:
  - java
  - pdf
  - testing
---

I’ve been wondering for some time if PDF is still a valid format. It’s “portable”, of course, but not in today’s meaning – it’s clearly not responsive. Like a fixed piece of paper transformed into a file. However, PDF still has many important use cases like storing invoices, reports or tickets. I spent a couple of years working on sophisticated PDF reports, and this year I even tried to test a process of generating invoices in some ad exchange system. I really wanted this system to be rock solid.

Of course there is no point in comparing a binary PDF file to an expected value. You can’t catch the exact differences in case of an error. I could create a PNG screenshot and compare it to the template, but I was a bit worried about the readability of such diff. A third way would be to verify the source HTML document used to render a PDF – but in fact, I was not interested in markup, but in an output data that landed inside a PDF.

Following my friend’s advice, I used another tool called [Apache PDFBox](https://pdfbox.apache.org/). This robust library allows performing different operations on PDF documents: creating, merging, splitting, signing, filling forms etc. We decided to extract plain text from a file. It’s like we used a *Select all* command, copy and paste the text into Notepad.

```java
PDDocument pdf = PDDocument.load(content);
PDFTextStripper stripper = new PDFTextStripper();
stripper.setAddMoreFormatting(true);
stripper.setSortByPosition(true);
stripper.setStartPage(0);
stripper.setEndPage(pdf.getNumberOfPages());
String plainText = stripper.getText(pdf);
pdf.close();

assertThat(plainText).isEqualTo(expected);
```

A PDF document consists of blocks which can be ordered in a way that we did not really expect. Anyone who tried copying a table from PDF to Notepad experienced this. Luckily, PDFBox tries to help us organizing the blocks and formatting the plain text dump.

We made a lot of test scenarios using the above solution and they did a really good job catching all the little bugs in the data. It was crucial to detect any mistakes because our system was preparing financial documents. Moreover, the test reports were very readable.

The only problem with the above method is that it does not test the layout correctness. To achieve that, we could extract only specified regions from the document. In such case we assume that a rectangle with `x1,y1,x2,y2` coordinates contain, for example, customer’s data:

```java
Rectangle2D region = new Rectangle2D.Double(x1, y1, x2 - x1, y2 - y1);
PDDocument pdf = PDDocument.load(content);
PDFTextStripperByArea stripper = new PDFTextStripperByArea();
stripper.addRegion("Contact region", region);
stripper.extractRegions(pdf.getPage(0));
String plainText = stripper.getTextForRegion("Contact region");
pdf.close();

assertThat(plainText).isEqualTo(expected);
```
