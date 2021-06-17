---
layout: post
title:  "How to encrypt a PDF document in PHP"
date:   2021-06-16 17:00:00 +0100
description: "Examples of encrypting a document with TCPDF, mPDF, Dompdf and PDFtk"
permalink: /2021/06/16/php-how-to-encrypt-a-pdf-document/
tags:
  - pdf
  - php
  - security
---

If your business uses [Portable Document Format](https://en.wikipedia.org/wiki/PDF) to send private and sensitive data like bank documents, you might need to use password protection. In this article you'll see how to encrypt PDFs with tools available for PHP.

## Types of encryption

To protect document contents, an encryption algorithm has to be used. PDF supports *symmetric ciphers* which use a password specified by the document creator to build an encryption key. The person who receives the document has to enter that password in order to decrypt the document.

<figure>
  <img src="/assets/pdf-password-protection.png" width="634" height="326" alt="Password window in Document Viewer (Ubuntu)">
  <figcaption>Document Viewer on Ubuntu asking for a password</figcaption>
</figure>

We have two algorithms to choose from, with different key lengths. The longer the encryption key used, the harder it is to crack the code:

1. [RC4](https://en.wikipedia.org/wiki/RC4). The first algorithm supported by PDF. Unfortunately it is perceived as **insecure** because multiple vulnerabilities were discovered. Still, it's the only algorithm implemented by most free PDF generators. Available key lengths are **40 and 128 bits.**
2. [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). This algorithm is approved even by the U.S. government to protect classifed information. There is no foreseeable possibility to crack the AES cipher in a reasonable time; with modern hardware it would take billions of years. If you receive password-protected bank documents, they're most likely encrypted with AES. Available key lengths are **128 and 256 bits.**

## User permissions

When encrypting a PDF document, you can specify two passwords. One of them is for you as the document *owner*, so you can perform any editing and printing tasks. You can also set a *user* password that gives limited access to the document.

You decide what privileges you give to other people. For example, you might disallow full quality prints, so that users have only a preview. You might disable editing, disassembling pages structure, filling forms, and so on.

It is however up to the PDF reader to enforce these rules. A hacker could implement their own reader to disobey the limitations. Once a document is decoded with a password, a reader has full access to it and can perform any operations.

In the examples below we will set separate owner and user password, but the latter is always optional.

## Encryption with TCPDF

[The TCPDF library](https://tcpdf.org/) is the only free tool I know which supports all ciphers, including the strongest 256-bit AES. Internally, TCPDF uses PHP's [OpenSSL](https://www.php.net/manual/en/book.openssl.php) and [Hash](https://www.php.net/manual/en/book.hash.php) extensions to perform encryption.

```php
$pdf = new TCPDF('P', 'mm', 'LETTER');
$pdf->SetProtection(
    ['print', 'modify', 'copy', 'annot-forms', 'fill-forms', 'extract', 'assemble', 'print-high'],
    'test123', 'test456', 3
);
$pdf->AddPage();
$pdf->writeHTML('<h1>Hello world</h1>');
file_put_contents('output.pdf', $pdf->Output('', 'S'));
```

The last argument to the `SetProtection()` method is a number specifying the algorithm and key length. The numbers start with `0` for a 40-bit RC4, and end with `3` representing 256-bit AES.

If you put an empty array as the first argument, no permissions will be granted for document users except for displaying it on a screen.

## Encryption with mPDF

[The mPDF library](https://github.com/mpdf/mpdf) is better in HTML rendering than TCPDF. Unfortunately it doesn't support such a wide range of encryption algorithms. You can only choose between 40-bit and 128-bit RC4 ciphers. The key length is specified as the fourth argument to the `SetProtection()` method below:

```php
use Mpdf\Mpdf;

$pdf = new Mpdf(['format' => 'LETTER', 'orientation' => 'P']);
$pdf->SetProtection(
    ['print', 'modify', 'copy', 'annot-forms', 'fill-forms', 'extract', 'assemble', 'print-highres'],
    'test123', 'test456', 128
);
$pdf->writeHTML('<h1>Hello world</h1>');
$pdf->Output('output.pdf', 'F');
```

## Encryption with Dompdf

[The Dompdf library](https://github.com/dompdf/dompdf) is quite good at rendering HTML and CSS code into a PDF. When it comes to encryption, it supports only the weak 40-bit RC4 cipher:

```php
use Dompdf\Dompdf;

$pdf = new Dompdf();
$pdf->getCanvas()
    ->get_cpdf()
    ->setEncryption('test123', 'test456', ['print', 'modify', 'copy', 'add']);
$pdf->loadHtml($html);
$pdf->render();
file_put_contents('output.pdf', $pdf->output());
```

Also, Dompdf supports only basic four permissions from an older PDF standard.

## Encryption with FPDF

[The FPDF library](http://www.fpdf.org/) does not have built-in encryption, but there's a separate [code snippet to implement 40-bit RC4.](http://www.fpdf.org/en/script/script37.php)

Setasign, the company behind FPDF, offers a [commercial library that supports encryption up to 256-bit AES.](https://www.setasign.com/products/setapdf-core/details)

## Encrypting an existing file with PDFtk

If your favorite PDF generator does not offer encryption, you can use [PDFtk Server](https://www.pdflabs.com/tools/pdftk-server/) to encrypt an existing file with 128-bit RC4. PDFtk does not support AES.

With PDFtk, protecting a file is as simple as running the command below in your terminal:

```
pdftk input.pdf output encrypted.pdf owner_pw test123 user_pw test456
```

## Summary

Document encryption is a good way to protect the document contents from being accessed by an unauthorized person. Banking documents are often sent via email and they could be stolen from a person's account. With strong encryption, it's not possible to read them.
