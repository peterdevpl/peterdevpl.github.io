---
layout: post
title:  "I moved my WordPress blog to Jekyll. Here's why and how"
date:   2021-02-26 20:00:00 +0100
permalink: /i-moved-my-wordpress-blog-to-jekyll-heres-why-and-how/
description: My story about moving a blog from WordPress to Jekyll - a static site generator.
redirect_from: /2021/02/26/i-moved-my-wordpress-blog-to-jekyll-heres-why-and-how/
tags:
  - blogging
---

**I remember the times around 2000 when most websites were static.** We edited them locally on our computers and then uploaded to an FTP server. There was plenty of free hosting services. Building your own site was very easy.

Then things became complicated. We were fascinated by the possibilities of PHP - a dynamic script interpreter, coupled with a MySQL database system. Of course such setup requires more server resources and page serving time is longer, but we were so thrilled we didn't care.

Today I know it was stupid to run the whole PHP ecosystem only to join some text files together. Until February 2021 my blog was still built with WordPress. If I was clever enough to design and use a **Static Site Generator** in 2000...

## Why use a Static Site Generator?

### No need for a backend

I no longer need a full LAMP stack to host my blog. This allowed me to ditch my Lightsail instance which costed me several bucks a month. All pages are generated upfront and then served as static files.

### I can code myself

WordPress and other Content Management Systems are good for people who can't code. They log into the admin section and write posts with a WYSIWYG editor.

I found the WordPress editor bad for technical writing. This is where **Markdown** comes into play. I get syntax highlighting for my code snippets out of the box.

### Free hosting

It's a lot easier to find a free static site hosting than a server with a fully functional backend. I'm using [GitHub Pages](https://pages.github.com/) which integrates Jekyll, so all I need to do is to push changes into the repository and the site is regenerated automatically. No additional CI/CD pipelines are needed.

Also, GitHub Pages allows you to use your custom domain and automatically provides a TLS certificate for that domain from [Let's Encrypt](https://letsencrypt.org/). Refer to GitHub documentation to learn more.

### Page loading speed

After moving to a static site, I noticed an increase in [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) from 70-80% to 100%. The site loads faster because there is no backend and because I have full control over all the stylesheets, scripts and images that are loaded.

### No cookies

Using a full backend usually comes with some cookies, for example for user session handling. I also had a Google Analytics script attached.

However, with cookies you are obliged to add that annoying and distracting warning for the EU. Moreover, a worldwide discussion about privacy is getting bigger every year. Some people stand against the so-called "surveillance capitalism", where by using "free" products we ourselves become a product.

I moved my site statistics to [Plausible.io](https://plausible.io/). It turns out you can do decent analytics without cookies.

## Building with Jekyll

The installation process and setting up a basic site with [Jekyll](https://jekyllrb.com/) is well documented. I will guide you step by step through how I adjusted the default setup to my needs. You are also invited to [browse my blog repository](https://github.com/peterdevpl/peterdevpl.github.io/).

Note that in order to host your site on GitHub Pages, the repository name must follow the format: *username*.github.io.

The initial commit consisted of just the default files and filling `_config.yml` with basic details:

```yaml
name: Piotr Horzycki
title: Piotr Horzycki - Java and PHP developer's blog
email:
description: >-
  Software engineer since 2008. Experienced with complex systems for payments, media, advertising and education.
  Been a scrum master and a team leader. I love fintech, data processing and SQL optimization. Sometimes I talk at meetups.
url: "https://peterdev.pl"
twitter_username: peterdevpl
github_username:  peterdevpl
theme: minima
```

After typing `jekyll serve` in the console, I was able to view my site at `http://localhost:4000`.

### Writing posts

I spent a couple of days reformatting over 30 posts using Markdown. I store all of them in the `_posts` directory. Names follow one pattern: `YYYY-MM-DD-slugged-post-title.markdown`. All files start with a *front matter* which contains metadata, for example:

```yaml
layout: post
title: "Picking a PHP tool to generate PDFs (2021 update)"
date: 2019-01-11 17:00:00 +0100
last_modified_at: 2021-01-10 17:00:00 +0100
description: "Comparison of HTML to PDF conversion tools: mPDF, TCPDF, Dompdf, wkhtmltopdf and Headless Chrome."
excerpt: I spent a lot of time working with different tools to generate PDF files, mainly invoices and reports. Some of these documents were really sophisticated, including multi-page tables, colorful charts, headers and footers. I tried generating documents by hand and converting HTML to PDF, or even LaTeX to PDF.
image: /assets/generating_pdf_files.jpg
permalink: /2019/01/11/picking-a-php-tool-to-generate-pdfs/
tags: pdf php
```

I don't always use all metadata, but in the example above I really need to tell people that the article has been updated. I also specify a short SEO description, excerpt to be published on the home page and an illustrative image.

During migration, I took great care to preserve all existing links and thus avoid trouble with redirecting pages indexed by search engines. For a permalink like above, Jekyll automatically generates the whole directory structure.

[Read more about the `jekyll-seo-tag` plugin](https://github.com/jekyll/jekyll-seo-tag) to know how to take care of your metadata and SEO.

### Changing permalinks and making redirects

By default, WordPress creates links that include the post date. I decided I no longer want to have the full date in my URLs. But **I can't just change all the links on my blog** all of a sudden. This would *destroy* my presence in the search engines, social media and people's bookmarks.

[The `jekyll-redirect-from` plugin](https://github.com/jekyll/jekyll-redirect-from) automatically creates redirects for me. All I need to after installing and enabling the plugin is a small change in the post's front matter:

```yaml
permalink: /picking-a-php-tool-to-generate-pdfs/
redirect_from: /2019/01/11/picking-a-php-tool-to-generate-pdfs/
```

Jekyll will compile the post to the new directory without a date prefix. In the old path, Jekyll creates a small HTML file which redirects the browser to the new link. That way, everyone having the old link can smoothly jump to the new version.

### Customizing post layout

By default, Jekyll provides a theme called Minima. You might customize it by either adjusting some configuration options, or copy-pasting the layout files into your blog project. Either way you have to inspect the [Minima's source code](https://github.com/jekyll/minima) to check the file and variable names. Be sure to browse the correct version of the repository.

I created my own `_layouts/post.html`, so it includes additional metadata I've been using in my articles:

{% raw %}
```html
---
layout: default
---
<article class="post h-entry" itemscope itemtype="http://schema.org/BlogPosting">
  <header class="post-header">
    {%- if page.image -%}
      <img src="{{ page.image }}" width="740" height="340" alt="Featured illustrative image" class="featured-image">
    {%- endif -%}
    <h1 class="post-title p-name" itemprop="name headline">{{ page.title | escape }}</h1>
    <p class="post-meta">
      {%- if page.last_modified_at -%}
          Last updated <time class="dt-modified" datetime="{{ page.last_modified_at | date_to_xmlschema }}" itemprop="dateModified">{%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}{{ page.last_modified_at | date: date_format }}</time>, first published
          <time class="dt-published" datetime="{{ page.date | date_to_xmlschema }}" itemprop="datePublished">
            {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
            {{ page.date | date: date_format }}
          </time>
        {%- else -%}
          <time class="dt-published" datetime="{{ page.date | date_to_xmlschema }}" itemprop="datePublished">
          {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
          {{ page.date | date: date_format }}
          </time>
      {%- endif -%}
    </p>
  </header>

  <div class="post-content e-content" itemprop="articleBody">
    {{ content }}
  </div>
</article>
```
{% endraw %}

The layout files are parsed by the [Liquid template engine](https://shopify.github.io/liquid/). Refer to its documentation to know the syntax.

### Customizing header and footer

I simplified both the header and footer by creating my own `_includes/header.html` and `_includes/footer.html` files. I initially copied the original Minima code and then adjusted it. I invited people to browse the blog's repository.

### Customizing home page

I wanted to remove the "Posts" header, show post excerpts and more metadata. First, I added `show_excerpts: true` to my `.config.yml`. Then I copy-pasted Minima `_layouts/home.html` and adjusted the posts list:

{% raw %}
```html
{%- for post in site.posts -%}
<li>
  {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
  <span class="post-meta">
    {%- if post.last_modified_at -%}
      {{ post.date | date: date_format }}, last updated {{ post.last_modified_at | date: date_format }}
    {%- else -%}
      {{ post.date | date: date_format }}
    {%- endif -%}
  </span>
  <h3>
    <a class="post-link" href="{{ post.url | relative_url }}">
      {{ post.title | escape }}
    </a>
  </h3>
  {%- if site.show_excerpts -%}
    {{ post.excerpt }}
  {%- endif -%}
</li>
{%- endfor -%}
```
{% endraw %}

### Linking social media accounts

The default theme, Minima, will embed links to your social media. You just need to specify account names in `.config.yml`. I already did this for Twitter and GitHub, but Minima also accepts other, including LinkedIn and Facebook:

```yaml
twitter_username: peterdevpl
github_username:  peterdevpl
linkedin_username: piotr-horzycki
facebook_username: piotr.horzycki
```

The social media icons are rendered inside `_includes/social.html` file which you can override if you really need to.

### Grouping posts by tags

In my WordPress instance I had all the tags under `/tag/something` URLs. In Jekyll, I can recreate that structure with `jekyll-archives` plugin. First I install it in the command line by typing `gem install jekyll-archives`. Then I go to `.config.yml` and set the plugin up:

```yaml
plugins:
  - jekyll-archives

jekyll-archives:
  enabled:
    - tags
  layouts:
    tag: tag
  permalinks:
    tag: '/tag/:name/'
```

In the configuration we mentioned a layout called `tag`. It will contain the markup needed to list all posts for a given tag. Let's create a file `_layouts/tag.html`:

{% raw %}
```html
---
layout: default
---

<h1>Tag: {{ page.title }}</h1>

<section class="main">
  <ul>
    {% for post in page.posts %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
</section>
```
{% endraw %}

The layout inherits from a default one, so there's no need to attach all the surrounding markup. ...

Now we need to add the list of tags to the post layout. I added the following code to the meta paragraph:

{% raw %}
```html
{%- if page.tags -%}
  •
  {% for tag in page.tags %}
    {% assign tag_slug = tag | slugify: "raw" %}
    <a href="/tag/{{ tag_slug }}/">#{{ tag }}</a>
  {% endfor %}
{%- endif -%}
```
{% endraw %}

The hash above just adds a hash sign, it's not a part of the syntax.

The `jekyll-archives` plugin can also list your posts by months. [See the full guide](https://github.com/jekyll/jekyll-archives)

### Setting ATOM feed

I have my blog aggregated in some lists, so I need to maintain either an RSS or ATOM feed. WordPress did this automatically under the path `/feed/`. Jekyll by default creates a `feed.xml` file in the root directory. I wanted to keep my old feed URL, so I did this in configuration:

```yaml
feed:
  path: /feed/index.xml
```

[More feed options](https://github.com/jekyll/jekyll-feed)

### Extending the style sheets

I needed some extra CSS rules for image figures and to change some colors. Minima uses SASS to write and compile style sheets. Let's start from `_config.yml`:

```yaml
sass:
  sass_dir: _sass
  style: compressed
```

Then I copy-pasted `_sass/minima.scss` and linked my two new files:

```scss
$very-light-grey: #F8F8F8;

@import
  "minima/base",
  "minima/layout",
  "minima/syntax-highlighting",
  "figures",
  "layout"
;
```

The first file, `figures.scss`, contains some image-related rules:

```scss
figure {
  text-align: center;
}

figcaption {
  color: $grey-color;
}

.featured-image {
  height: auto;
  margin-bottom: 1ex;
}
```

The `layout.scss` provides just some eye-candy:

```scss
.site-header, .site-footer {
   background: $very-light-grey;

   .built {
      color: $grey-color-dark;
      font-size: 90%;
      margin-bottom: 0;
   }
}
```

Jekyll automatically compiles the style sheets, so I don't need any other tools.

### Using a custom domain

My blog has been working under `https://peterdev.pl`, so I wanted to keep it that way. I already had around 200 visitors from search engines every day.

[GitHub Pages allows setting a custom domain.](https://docs.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site) The official guide is a bit messy, so I had to do some more digging and experiments. I decided to use only the apex domain (`peterdev.pl`), as the `www.` subdomain didn't work. I logged in to my domain registrar and set my A and CNAME records like this:

A     | peterdev.pl.     | 185.199.111.153
A     | peterdev.pl.     | 185.199.110.153
A     | peterdev.pl.     | 185.199.109.153
A     | peterdev.pl.     | 185.199.108.153
CNAME | www.peterdev.pl. | peterdevpl.github.io.

I also had to open my GitHub blog repository, go to the `Settings` page, set the `Custom domain` to `peterdev.pl` and tick `Enforce HTTPS`. As the DNS changes might take several hours to propagate, GitHub will initially complain about bad DNS configuration. You have to wait until GitHub gets your new DNS records and generates the TLS certificate. Then your blog should work under your domain and with HTTPS enforced.

## I wish I have done this earlier

Keep it simple! Don't use over-engineered solutions for simple problems. I wish I had my blog optimized as a static site from day one and haven't paid for additional hosting.

Jekyll has many alternatives, and so does GitHub. For me, this combo works perfectly fine, but you're free to discover other options.

**You can also browse the [entire source repository for my blog.](https://github.com/peterdevpl/peterdevpl.github.io/)**
