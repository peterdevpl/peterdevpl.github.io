---
layout: post
title:  "Is Portable Document Format obsolete?"
date:   2019-09-06 17:00:00 +0100
description: In the age of responsive web design, PDFs are still very common. When is it a proper format for your data?
permalink: /2019/09/06/is-portable-document-format-obsolete/
tags:
  - html
  - pdf
---

I spent two years on a project which aimed to create highly sophisticated PDF reports for teachers. These automatically generated reports contained bar chars, pie charts, line charts, a lot of tables and everything else. It worked: end users were very happy about the new reports system. But still, something keeps bothering me.

## Making PDFs can be tough. Do we still need them?

There are many ways you can generate PDF files. I started from a Java class which calculated every table cell dimensions, position of every text line etc. Then I used PHP libraries like FPDF and TCPDF. The latter had a very simple HTML parsing engine. Then I came across `Zend_Pdf` and kept using it until I stumbled upon `wkhtmltopdf`. Yeah, that was a blast! Someone put together a WebKit rendering engine and a PDF printer.

But even wkhtml was not prepared for all the challenges that my client had for me. Dynamic headers and footers. Sophisticated charts which I prepared as SVG files. Errors with dividing tables across multiple pages (with table headers of course). Lack of dynamic font size adjustment depending on the text amount. Trouble with applying custom fonts.

Chrome 59 offered printing to PDF via headless mode. Having recent Chrome rendering engine is great, but I lost some features available in wkhtmltopdf (like these crazy page headers and footers).

You can already see that generating proper PDF files might require a lot of tweaking and quirks. What if we don’t need PDFs at all? In the era of portable devices coming in all shapes and sizes, do we still have to limit our documents in an A4 or any other paper format?

## Is PDF always a right choice for our project?

Let’s think about the business needs for a while. The system I mentioned was designed for teachers who got used to print everything on a daily basis. I’m not sure how it works in other countries, but Polish teachers print a lot of stuff. They download a PDF, then print and put on their desks to discuss with pupils and their parents. That’s their habit.

I worked on a similar reporting system designed specifically for parents. The initial business decision was to copy the same well-known PDF system and do some adjustments. I asked my client: hey, do we really need to spend a lot of time fine-tuning PDFs again? Do our end users need PDFs? They have different habits, they have smartphones and tablets. They are not going to print our reports. They want a modern, responsive web app.

That’s how I convinced my client to drop PDFs in the new system. I made development a lot faster and easier. We made a modern frontend layer using ReactJS and everyone was happy with the result.

Do we still need static PDF layouts for invoices, order confirmations, tickets, magazines, official documents, reports?

In Poland, we have electronic train tickets that can be downloaded on a smartphone. It is an ordinary PDF file with all the ticket details. But all we have to do is to show a QR code during ticket control. The same applies to concert tickets. Why use PDF if a simple HTML would be enough?

## Further reading

[Will PDF files become obsolete in 5-10 years?](https://www.quora.com/Will-PDF-files-become-obsolete-in-5%E2%80%9310-years)

{% include book.html %}
