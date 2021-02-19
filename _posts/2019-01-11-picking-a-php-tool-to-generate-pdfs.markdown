---
layout: post
title:  "Picking a PHP tool to generate PDFs (2021 update)"
date:   2019-01-11 17:00:00 +0100
last_modified_at: 2021-01-10 17:00:00 +0100
description: "Comparison of HTML to PDF conversion tools: mPDF, TCPDF, Dompdf, wkhtmltopdf and Headless Chrome."
excerpt: I spent a lot of time working with different tools to generate PDF files, mainly invoices and reports. Some of these documents were really sophisticated, including multi-page tables, colorful charts, headers and footers. I tried generating documents by hand and converting HTML to PDF, or even LaTeX to PDF.
permalink: /2019/01/11/picking-a-php-tool-to-generate-pdfs/
tags:
  - pdf
  - php
---

> TL;DR For HTML to PDF conversion, use [Dompdf](https://github.com/dompdf/dompdf) library if you don’t need CSS Flexbox or Grid layouts. Neither Dompdf, mpdf, TCPDF nor wkhtmltopdf supports Flexbox or Grid. Use [Google Chrome in headless mode](https://developers.google.com/web/updates/2017/04/headless-chrome#create_a_pdf) if you need modern CSS rules. Consider [browserless](https://docs.browserless.io/docs/pdf.html).

I spent a lot of time working with different tools to generate PDF files, mainly invoices and reports. Some of these documents were really sophisticated, including multi-page tables, colorful charts, headers and footers. I tried generating documents by hand and converting HTML to PDF, or even LaTeX to PDF.

I know how hard it is to choose between a multitude of libraries and tools, especially when we need to do a non-trivial job. There is no silver bullet; some tools are better for certain jobs and not so good for other jobs. I will try to sum up what I’ve learned through the years.

## Two ways of creating a PDF file

**A PDF file contains a set of objects which make a document**, like pieces of text, images, lines, forms, fonts, and so on. So creating a PDF is an act of putting all these pieces together in a proper order and layout. Most objects utilize a subset of PostScript commands, so you can even [write them in your text editor](https://brendanzagaeski.appspot.com/0004.html).

One way is to **create these objects “by hand”**: we add every line of text separately, we draw all tables manually, calculating cell widths and spacings on our own. We must know when to split longer contents into multiple pages. This approach requires a lot of manual work and very good programming skills, so we don’t end up with a spaghetti code where it is hard to find any meaningful logic between all the drawing commands.

Another way is to **convert one document, for example HTML, LaTeX or PostScript into PDF**.

We used [LaTeX](https://www.latex-project.org/) for an education app which allowed composing tests for students from existing exercises prepared by professionals. Since LaTeX was a primary tool for our editors, it was natural for us to convert their scripts straight to PDF.

**Converting HTML to PDF** is way more complex as today’s web standards are having more and more features, just to mention CSS Flexbox or Grid layouts. Let’s see what we can do.

## Native PHP libraries

My first experience was with native PHP libraries where you had to do most things by hand, like placing text in proper positions line by line, drawing rectangles, calculating table cells, and so on. It was quite fun at the time, but creating more robust documents turned out to be very hard. We used [FPDF](http://www.fpdf.org/) and [ZendPdf](https://github.com/zendframework/ZendPdf) libraries (the latter is discontinued).

At some point, I ended up maintaining multiple-page, sophisticated school reports with tables and charts rendered by ZendPdf. Business wanted to add even more types of reports. I decided to rewrite all reports as HTML documents with stylesheets and then try to make PDFs from that.

**There are three PHP libraries capable of parsing HTML/CSS and transforming that to PDF:**

* [mPDF](https://github.com/mpdf/mpdf)
* [TCPDF](https://github.com/tecnickcom/TCPDF) (will be replaced some day by [tc-lib-pdf](https://github.com/tecnickcom/tc-lib-pdf))
* [Dompdf](https://github.com/dompdf/dompdf)

Rendering HTML and CSS certainly isn’t easy, so you cannot expect these libraries to provide the same output you’re seeing in Firefox or Chrome. However, **for simple layouts and formatting they should be enough**. Plus is that you still do not depend on any external tools – just plain PHP!

To give you some idea of what to expect from above libraries, I compiled **a comparison of an invoice renderings**. These three pictures are made from the same HTML 5 source which utilizes CSS Flexbox to position “Seller” and “Buyer” sections next to each other. It has also some table formatting:

Google Chrome (reference image)
TCPDF 6.3.5
mPDF 8.0.10
Dompdf 1.0.2

As you can see, **none of the PHP libraries understood CSS Flexbox**. mPDF and TCPDF had some problems with painting the table. **Dompdf performed the best** and I’m pretty sure that making the “Seller” and “Buyer” sections the old-school way, like `float` or `<table>` would be enough to have a proper result.

## External CLI tools

Native PHP solutions were not enough for me, so I decided to use an external tool backed by a fully functional, WebKit rendering engine. My employer was already using [wkhtmltopdf](https://wkhtmltopdf.org/) which supports everything I needed: SVG images, multi-page tables, headers and footers with page numbers and section names, automatic bookmarks. Having old reports rewritten to HTML and CSS, I was able to implement all the new features requested by the business.

`wkhtmltopdf` certainly isn’t bug-free; for example, I had some issues with repeating table headers on consecutive pages. Also, upgrading from 0.12.3 to 0.12.4 broke my document layout which used dynamic headers and footers, so I had to go back to the old version.

Then I got familiar with [PhantomJS](http://phantomjs.org/), which was used mainly to conduct automatic browser tests in a headless mode (without the browser window). It could also capture PNG and PDF screenshots. PhantomJS used a newer version of the WebKit engine. However, the project is suspended now.

Almost a year before the suspension of PhantomJS, [Google announced that Chrome can run in a headless mode](https://developers.google.com/web/updates/2017/04/headless-chrome). This means you can utilize the latest Blink rendering engine to convert HTML/CSS to PDF from your command line. This is perfect for rendering really complex documents utilizing latest web standards. The document looks exactly the same in your browser and in the final PDF file which makes development a lot easier.

## Connecting PHP with external tools

The easiest way would be to execute an external tool as a shell command. You can do it with PHP functions like `shell_exec` or `proc_open`, but it’s not very convenient.

I recommend using [symfony/process](https://symfony.com/doc/current/components/process.html) library and utilize standard streams whenever applicable. A process should accept input HTML through STDIN and send the resulting PDF via STDOUT. It can also produce some errors through STDERR. It might turn out that you won’t need any temporary files to do the job.

There are also several wrapper libraries, like [phpwkhtmltopdf](https://github.com/mikehaertl/phpwkhtmltopdf) or [KnpLabs/snappy](https://github.com/KnpLabs/snappy).

For Chrome, consider using [Browserless](https://www.browserless.io/). You can choose between a free Docker image with pre-configured Chrome with dependencies, or a paid SaaS platform to convert your HTMLs to PDF. With the Docker image, it is really easy to [send HTML and receive PDF via HTTP](https://docs.browserless.io/docs/pdf.html).

## Conclusion

There is a wide choice of PHP libraries and external tools which can be used to dynamically create PDF files. You should choose a combination which suits your business needs. **For simple documents, you don’t need a complex rendering engine**. Save disk space, CPU and RAM!

Please also remember that many tools are developed by the Open Source community and receive little commercial support. They can be abandoned at any time or they might not support newest PHP version from day one (which can impede migrating the rest of your app). And your dependencies have dependencies too, so take a look at `composer.json` when picking a library.

And if your favorite Open Source tool does not do everything you need properly – maybe try contributing? It’s a *community*, after all.
