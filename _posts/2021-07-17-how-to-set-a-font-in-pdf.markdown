---
layout: post
title:  "How to set a font in a PDF document"
date:   2021-07-17 15:00:00 +0100
description: "How to set a font in a PDF document with mPDF, Dompdf, wkhtmltopdf, Chrome, WeasyPrint and Prince"
excerpt: "In this article, you will learn how to set custom fonts when converting HTML to PDF. We will cover several conversion tools, including Headless Chrome, WeasyPrint, Prince, wkhtmltopdf, mPDF and Dompdf."
permalink: /2021/07/17/how-to-set-a-font-in-pdf/
tags:
  - pdf
  - php
---

In this article, you will learn how to set **custom fonts when converting HTML to PDF.** We will cover several conversion tools, including [Headless Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome), [WeasyPrint](https://weasyprint.org/), [Prince](https://www.princexml.com/), [wkhtmltopdf](https://wkhtmltopdf.org/) and PHP libraries: [mPDF](https://github.com/mpdf/mpdf) and [Dompdf](https://github.com/dompdf/dompdf).

## Some theory about fonts and text

Before we start, there are some terms you should familiarize with.

Most documents are based on text. To build a piece of text you need characters that will make letters and words.

A *character set* defines mappings between numeric codes and characters: letters, digits, symbols, and so on. For example, in the [ASCII table](https://www.asciitable.com/), the decimal number 65 represents a Latin letter A. This is an abstract representation; we still don't know how this letter should be drawn on screen or printed.

An *encoding* specifies how the character codes will be represented as bytes. For ANSI this is simple: a byte value 65 (decimal) is equal to ASCII code 65, which represents capital letter A. However, if a character set exceeds 256 possible values of a single byte, we dive into the world of *multi-byte encodings*. The most popular ones are UTF-8, UTF-16, UTF-32, UCS-2 and UCS-4 for the Unicode standard.

A *font* is a set of *glyphs* - readable characters and other symbols that represent a character set. A font data file contains either bitmaps or vectors that make up all the character shapes.

**The first thing you need to properly render your HTML code to PDF is a character set declaration:**

```html
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    ...
  </body>
</html>
```

Having these basics described, we can start using fonts and typing!

## Picking a proper font

To use a custom font, first you have to choose one that covers **all characters you need in your document or its part.** This should be common sense, but sometimes we (or the client) forgets about it.

For example if you pick a fancy header font and your language includes non-Latin characters (accents, umlauts, ogonki, Cyrillic alphabet etc.), check if the font contains glyphs for them! Either use a website that allows testing fonts or download the font files and try them in some text editor or graphics program.

Usually, there are no "one-size-fits-all" solutions. Some fonts do not have an "italic" or "bold italic" versions on purpose. Some fonts contain only uppercase letters (capitals). Other fonts, like fancy handwriting-like ones, are not readable in small sizes.

## Font types supported by PDF

The most common font file formats are [OpenType](https://en.wikipedia.org/wiki/OpenType), [TrueType](https://en.wikipedia.org/wiki/TrueType) and [Type 1](https://en.wikipedia.org/wiki/PostScript_fonts#Type_1). They differ in features and the way of describing shapes. All of them can be used in a PDF document.

The so-called **"web fonts"** are usually compressed with a [WOFF2 format](https://www.w3.org/TR/WOFF2/) which is not supported by PDF. [Google Fonts](https://fonts.google.com/), a popular web font provider, fortunately offers a "Download family" feature which gives you the full TrueType archive.

However, if you only have a WOFF2 font file, you can still convert it to TrueType or OpenType. Either use an online tool, or the Linux terminal:

```
sudo apt install fontforge woff2
woff2_decompress font.woff2
```

## Selecting a font in CSS

Let's remind ourselves how to pick a font in CSS. The most basic syntax looks like this:

```css
body {
  font-family: Verdana, Arial, sans-serif;
}
```

The example above means that we prefer the Verdana font, but in case if it's not available we recommend substituting it either with Arial or any sans-serif font. We depend only on fonts available in a certain system. Every OS has a basic set of fonts, but you can also install your own.

Moreover, every PDF reader provides standard Type 1 fonts, including Times-Roman, Helvetica, Courier and Symbol.

You might want to use a custom font in your document without installing it globally in the operating system. In the example below, we import a font file and assign a local name `Lato`. We declare this is a normal (not italic) font of a regular weight:

```css
@font-face {
  font-family: 'Lato';
  font-style: normal;
  font-weight: normal;
  src: url('file:///path/to/my/project/lato.ttf') format('truetype');
}

body {
  font-family: Lato;
}
```

> The `@font-face` syntax works fine with any Chromium-based tools, and also WeasyPrint and Prince. Other tools make selecting a font a bit harder.

## Providing a font to wkhtmltopdf

For security reasons, wkhtmltopdf blocks any access to remote font files. It cannot even read a font file from a local drive.

To pick a custom font, we will use a data URL trick. First we have to encode the font file with [Base64](https://en.wikipedia.org/wiki/Base64). We can use either the PHP function `base64_encode()`, the Linux console command `base64` or any Base64 encoder available online.

Then we copy the encoded file contents and paste into the CSS:

```css
@font-face {
  font-family: 'CaslonItalic';
  src: url(data:font/truetype;charset=utf-8;base64,PASTE_IT_HERE) format("truetype");
}

body {
  font-family: CaslonItalic;
}
```

Because an encoded font file can be very long, it's more convenient to move the `@font-face` declaration to a separate CSS file and then use `@include` to attach it to the main stylesheet. You can decide if you want to include that encoded file in your repository, or generate it on-demand in some build script.

## Providing a font to Dompdf

The Dompdf PHP library has its internal font metrics engine which incorporates local caching. The mechanism is cumbersome because you have to manually register the font before using it.

> Below, I assume that you've installed Dompdf with [Composer](https://getcomposer.org/), hence the `vendor` directory.

This can be done with a `load_font.php` script which is available in the [dompdf/utils](https://github.com/dompdf/utils) package. Since it would require to copy another repo to the `vendor/dompdf/dompdf` directory, I don't really like this method.

Another way is to extend your PDF rendering code. During the first round, **Dompdf will create cache files** in the `vendor/dompdf/dompdf/lib/fonts` directory - which means your script must have **write access** there. Next time, those cached resources will be used to embed the font in a PDF:

```php
use Dompdf\Dompdf;
use Dompdf\Options;

$fontDirectory = '/home/someuser/fonts';

$options = new Options();
$options->setChroot($fontDirectory);

$pdf = new Dompdf($options);
$pdf->getFontMetrics()->registerFont(
    ['family' => 'CaslonItalic', 'style' => 'italic', 'weight' => 'normal'],
    $fontDirectory . '/CaslonItalic.ttf'
);
$pdf->loadHtml($html);
$pdf->render();
file_put_contents('output.pdf', $pdf->output());
```

The `setChroot()` call is necessary for security purposes, so that Dompdf won't access any system files.

Note that when adding a font file you must specify its corresponding style and weight.

## Setting a custom font in mPDF

mPDF has a decent documentation which explains a lot of nuances related to [international font handling.](https://mpdf.github.io/fonts-languages/fonts-in-mpdf-7-x.html)

To use your own font you have to register it. There is one major drawback: you have to invent a **font family name that's all lowercase and without any spaces** nor other special characters. So instead of `font-family: 'DejaVu Sans'` you have to enter `font-family: dejavusans`.

You can register as many font directories as you need. Moreover, you'll need a **temporary directory to store font cache.** By default it's `vendor/mpdf/mpdf/tmp/mpdf/ttfontdata` (assuming you've installed mPDF with Composer) and your script must have write permissions for that. Fortunately you can set another cache path:

```php
use Mpdf\Config\ConfigVariables;
use Mpdf\Config\FontVariables;
use Mpdf\Mpdf;

$fontDirectory = '/home/someuser/fonts';

$defaultConfig = (new ConfigVariables())->getDefaults();
$fontDirs = $defaultConfig['fontDir'];

$defaultFontConfig = (new FontVariables())->getDefaults();
$fontData = $defaultFontConfig['fontdata'];

$mpdf = new Mpdf([
    'fontDir' => \array_merge($fontDirs, [
        $fontDirectory,
    ]),
    'fontdata' => $fontData + [
        'caslon' => [
            'I' => 'CaslonItalic.ttf',
        ],
    ],
    'tempDir' => $fontDirectory . '/tmp',
]);
$mpdf->WriteHTML($html);
$mpdf->Output('output.pdf', 'F');
```

When registering font files, you have to declare their style with `R`, `B`, `I` and `BI` identifiers, corresponding to "regular", "bold", "italic" and "bold italic" styles, respectively.
