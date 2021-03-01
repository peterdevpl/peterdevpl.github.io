---
layout: post
title:  "PHP: How to take a screenshot of a PDF page"
date:   2021-01-28 17:00:00 +0100
permalink: /2021/01/28/php-how-to-take-a-screenshot-of-a-pdf-page/
description: A guide on converting PDF files to PNG and JPEG using ImageMagick, GhostScript, Poppler or Inkscape. Choose the best solution for you!
tags:
  - php
  - pdf
---

If your application allows **uploading PDF files**, it’s likely that you need to prepare **screenshots or thumbnails** for these documents – at least the first page.

You can’t do this with a pure PHP setup. You’re going to need an **external application to read PDF and save an image**, like [ImageMagick](https://imagemagick.org/), [GhostScript](https://www.ghostscript.com/index.html), [Poppler](https://poppler.freedesktop.org/) or [Inkscape](https://inkscape.org/). Before you start coding, check which one is installed on your server.

Sometimes you might need to check how different tools work for your documents. There can be slight differences in font rendering, handling alpha channel in images, speed and output file size.

**For all cases we’re going to use the [Symfony Process library](https://symfony.com/doc/current/components/process.html)** to safely call external commands. Simply run `composer require symfony/process` in your project.

## Using ImageMagick

ImageMagick has a handy tool called `convert`. Under the hood, it uses GhostScript to parse a PDF file. The simplest usage below extracts the first page of a PDF file and saves it as PNG:

```php
use Symfony\Component\Process\Process;

$process = new Process([
    'convert',
    'input.pdf[0]',
    'output.png'
]);
$process->run();
if (!$process->isSuccessful()) {
    die('Error');
}
```

This code will extract the first page from `input.pdf` and save it as `output.png`. Note that pages in a document are zero-indexed. You can convert multiple pages if you wish.

The Process constructor takes an array of command-line arguments. The first element is always the command’s name or path to a program. I decided to put every argument in a separate line for clarity.

You might want to **adjust some options for ImageMagick**. For example, `-alpha off` and `-background white` will always set a **solid white background** even if the input document has a transparent background. With `-density 200` you can increase the resolution:

```php
use Symfony\Component\Process\Process;

$process = new Process([
    './convert',
    '-alpha', 'off',
    '-background', 'white',
    '-density', '200',
    'input.pdf[0]',
    'output.png'
]);
$process->run();
```

You can also create a JPEG thumbnail, for example 150 pixels wide with a quality set to 90%:

```php
use Symfony\Component\Process\Process;

$process = new Process([
    'convert',
    '-alpha', 'off',
    '-background', 'white',
    '-resize', '150',
    '-quality', '90',
    'input.pdf[0]',
    'thumbnail.jpg'
]);
$process->run();
```

**If you want to operate on variables and not on real files,** you can use STDIN and STDOUT to deliver the PDF and receive an image. Enter a hyphen (-) instead of the input and output file names, then supply custom input and retrieve output from the process:

```php
use Symfony\Component\Process\Process;

$process = new Process([
    'convert',
    '-alpha', 'off',
    '-background', 'white',
    '-[0]',
    'png:-'
]);
$process->setInput($pdf);
$process->run();

if (!$process->isSuccessful()) {
    echo $process->getErrorOutput();
    die('Error');
} else {
    $png = $process->getOutput();
}
```

[More examples can be found on the ImageMagick site.](https://imagemagick.org/script/convert.php)

## Using GhostScript

You might want to interact with Ghostscript directly, especially if for some reason ImageMagick is not installed or you need to fine-tune some rendering details:

```php
use Symfony\Component\Process\Process;

$process = new Process([
    'gs',
    '-dFirstPage=1',    // process only 1st page
    '-dLastPage=1',
    '-dNOPAUSE',        // don't pause after processing a page
    '-dBATCH',          // don't run the interpreter
    '-r144',            // resolution: 144 pixels per inch
    '-q',               // surpress messages
    '-sDEVICE=png16m',  // 24-bit PNG without alpha channel
    '-sOutputFile=test.png',
    'input.pdf'
]);
$process->run();
```

Ghostscript is a Postscript interpreter. By default, it offers a console and stops after each page, so we’re using some options to change that behavior. We chose a 24-bit PNG here, but there are other formats available: `pngalpha` or `jpeg` for example. Run `gs -h` in console to see a full list of available formats (devices).

[More Ghostscript command-line options](https://www.ghostscript.com/doc/current/Use.htm#Options)

## Using Poppler

There is a nice set of PDF tools called Poppler-Utils. One of them, `pdftocairo`, can convert a PDF to PNG, JPEG, TIFF, SVG or EPS. Usage is very simple:

```php
use Symfony\Component\Process\Process;

$process = new Process([
    'pdftocairo',
    '-png',
    '-singlefile',
    'input.pdf',
    'output'
]);
$process->run();
```

See [pdftocairo man page](http://manpages.ubuntu.com/manpages/trusty/man1/pdftocairo.1.html) for more options.

## Using Inkscape

Some people report Inkscape as the best application for exporting PDF files to bitmaps. This robust vector graphics editor can also be used in the command line:

```php
use Symfony\Component\Process\Process;

$process = new Process([
    'inkscape',
    'input.pdf',
    '--export-dpi=600',
    '--export-area-page',
    '--export-background=#FFFFFF',
    '--export-type=png',
    '--export-filename=output.png'
]);
$process->run();
```

By default, the background is transparent, so I explicitly requested a white background. Also, instead of `--export-area-page` you might want to use `--export-area-drawing` to get only the contents and not a full page.

You can use the `--pipe` switch to make Inkscape read data from STDIN. If you omit the `--export-filename` option, the output will be sent to STDOUT.

Refer to the [Inkscape man page](https://inkscape.org/doc/inkscape-man.html) for more options.

## Further reading

Check out [this StackOverflow thread to see more ideas on how to convert a PDF file to PNG.](https://stackoverflow.com/questions/653380/converting-a-pdf-to-png)

**Other articles on my blog:**

* [Picking a PHP tool to generate PDFs (2021 update)]({% post_url 2019-01-11-picking-a-php-tool-to-generate-pdfs %})
* [Picking a PHP tool to read and manipulate PDF files]({% post_url 2019-01-25-picking-a-php-tool-to-read-and-manipulate-pdf-files %})
