---
layout: post
title:  "Picking a PHP tool to read and manipulate PDF files (2021 update)"
date:   2019-01-25 17:00:00 +0100
last_modified_at: 2021-03-01 21:00:00 +0100
description: Extracting text and metadata from PDF, editing PDF files, adding stamps, extracting images, making screenshots. Updated for 2021.
excerpt: Today we will browse possibilities to read and edit existing PDF files.
image: /assets/reading_pdf_files.jpg
permalink: /2019/01/25/picking-a-php-tool-to-read-and-manipulate-pdf-files/
tags:
  - pdf
  - php
---

> **TL;DR** For simple PDF text and metadata extraction, use [pdfparser](https://github.com/smalot/pdfparser). For advanced options, try [pdftotext](http://manpages.ubuntu.com/manpages/bionic/man1/pdftotext.1.html) and [pdfinfo](http://manpages.ubuntu.com/manpages/bionic/en/man1/pdfinfo.1.html) from [Poppler](https://poppler.freedesktop.org/). To join or split PDF files, encrypt them or apply watermarks, use [pdftk](https://www.pdflabs.com/docs/pdftk-man-page/). To make a JPEG or PNG screenshot of a PDF, use [ImageMagick](http://www.imagemagick.org/discourse-server/viewtopic.php?t=31313) or [pdftocairo](http://manpages.ubuntu.com/manpages/bionic/en/man1/pdftocairo.1.html).

<aside class="book-ad">
  <a href="https://leanpub.com/mastering-pdf-with-php">
    <img src="https://d2sofvawe08yqg.cloudfront.net/mastering-pdf-with-php/small?1620897108" width="170" height="220" alt="Mastering PDF with PHP book cover"><br>
    Struggling with PDFs?<br>
    Get a <strong>free sample</strong> of&nbsp;my&nbsp;book!
  </a>
</aside>

[In the previous article](/2019/01/11/picking-a-php-tool-to-generate-pdfs/) I described several tools that can be used together with PHP to create PDF files. Back then, the choice was not easy and we had a lot of criteria to consider while picking the best tool. Today we will browse possibilities to read and edit existing PDF files.

## Native PHP libraries

Again, we will start from checking if there are any PHP libraries to manipulate PDF files without depending on external binary tools.

### pdfparser

There is an interesting library called [smalot/pdfparser](https://github.com/smalot/pdfparser). It has over 1500 stars on GitHub. It parses a PDF file into an array of document objects which is further processed to get what we need.

The library is convenient as it supports both parsing an existing file or a string with PDF data. **It allows you to extract metadata and plain text from a document** along with other objects (images, fonts). However, encrypted files are not yet supported. You can test the library at its [demo page](https://www.pdfparser.org/demo).

```php
$parser = new Smalot\PdfParser\Parser();
$document = $parser->parseFile('test.pdf');

// creator, date of creation, number of pages etc.
print_r($document->getDetails());

// text dump
echo $document->getText();
```

[smalot/pdfparser](https://github.com/smalot/pdfparser) has commercial support from Actualys.

### tc-lib-pdf-parser

[This is a library made by the creator of TCPDF](https://github.com/tecnickcom/tc-lib-pdf-parser), a well-known library generating PDF files. This parser draws less interest than the first one, though the author has over 15 years of experience handling PDFs.

You can compare both libraries by parsing different documents. They can differ especially in terms of processing corrupted files.

### FPDI

I got familiar with this library when I received a bug report for a watermarking module in some e-book system. The module received a PDF, parsed it using FPDI, generated a watermark with FPDF and stamped it over all pages.

The problem is that the *free version of FPDI supports only PDF version 1.4 and below*. To support higher document versions, you have to buy a full library. And that’s what the bug report was about. We decided to switch to another tool, `pdftk`, which is described below.

## Command-line tools

The first command-line tool I played with was [pdftk](https://www.pdflabs.com/docs/pdftk-man-page/). I used it to **join separate documents into one, apply watermarks and extract basic metadata**, like a number of pages. It supports all PDF formats unlike FPDI library. The only thing that’s missing is a text extraction feature.

The need to extract plain text from a document led me to the [Apache PDFBox library](https://pdfbox.apache.org/). It is written in Java and, as I described before, it offers some very nice features. However, in the PHP world we can only access a [CLI wrapper](https://pdfbox.apache.org/2.0/commandline.html) for that library which has a limited set of options.

Later I discovered the Poppler library, [which is said to fully support the ISO 32000-1 standard for PDF](https://www.fsf.org/blogs/community/gnu-pdf-project-leaves-high-priority-projects-list-mission-complete). This C++ library can be accessed via dedicated CLI tools – [poppler-utils](https://en.wikipedia.org/wiki/Poppler_(software)#poppler-utils), which we can run from PHP. For example, the `pdftotext` tool gives a lot of control over the **plain text dump** – you can even preserve a proper document layout while rendering, or crop the document to a specified region. Also, `pdfinfo` provides comprehensive **information about a file**, like page format, encryption type etc. You can use it to extract JavaScript too.

Sometimes you might want to **create a PNG or JPEG screenshot of a document**. You can do it with `pdftocairo` from Poppler, or use ImageMagick’s `convert`. At the time of writing, there are no native PHP libraries to render a PDF.

## Wrappers

For `pdftk`, check out this library: [mikehaertl/php-pdftk](https://github.com/mikehaertl/php-pdftk).

PDFBox CLI can be accessed via [schmengler/PdfBox](https://github.com/schmengler/PdfBox).

Imagemagick and Ghostscript are the basis for [spatie/pdf-to-image](https://github.com/spatie/pdf-to-image) wrapper.

Poppler has several PHP wrapper libraries:

* [spatie/pdf-to-text](https://github.com/spatie/pdf-to-text) only allows to extract text from a PDF. It requires an input PDF to exist in the file system. The library does not wrap additional input arguments, so you have to specify them manually.
* [ncjoes/poppler-php](https://github.com/ncjoes/poppler-php): a library supposed to wrap all `poppler-utils`, but at the moment `pdftotext` is still unsupported. Also, this library is not very convenient as it forces you to choose an output directory for a file (it does not return processed data as string).

In fact, these two libraries are wrappers to a wrapper, since `poppler-utils` are just a collection of CLI wrappers for the Poppler C++ library 😉

## Which to pick? Native or CLI?

There are a couple of basic considerations.

Native PHP libraries should work independently from the host environment. They are a lot easier to set up and update. The only depedency tool you use is Composer.

CLI tools, especially these written in C/C++, might be faster and use less memory. However I don’t have strict evidence at the moment. Maybe all the optimizations that came with PHP 7 will make this point obsolete. Also, I believe that C/C++ tools have a wider audience and thus might receive more community support.

You should pick a tool that’s best for your specific requirements. Most tools will do a decent job while simply rendering an unencrypted PDF to an image or some plain text. But if you need to have more control on the output file structure or you want to process encrypted documents, `poppler-utils` will be a good choice.

Sometimes it occurs to me that many developers are just reinventing the wheel, especially when it comes to a multitude of PDF processing libraries for PHP. The Portable Document Format has almost seven hundred pages of specification. We are all struggling with the same processing issues. That’s why I rather prefer to choose the best tools in different technologies and connect them with interfaces rather than doggedly sticking to a single technology.

Check out the [List of PDF software](https://en.wikipedia.org/wiki/List_of_PDF_software) at Wikipedia.

### See also

* [Picking a PHP tool to generate PDFs (2021 update)]({% post_url 2019-01-11-picking-a-php-tool-to-generate-pdfs %})
* [PHP: How to take a screenshot of a PDF page]({% post_url 2021-01-28-php-how-to-take-a-screenshot-of-a-pdf-page %})
