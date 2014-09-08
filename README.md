# FlatPress example site

## Introduction

[FlatPress](https://github.com/diegolagoglez/flatpress) is a really simple CMS with no database nor any dynamic language.

This is a example site repository to publish with it.

All entries are from my old and disapeared blog.

## Directory layout

* `contents`: Directory for site contents.
    * `articles`: Articles written in Markdown.
    * `aside`: Markdown files to show in sidebar (aside).
    * `pages`: Pages written in Markdown.
* `fonts`: Server side fonts.
* `resources`: Resources like templates.
* `static`: Static contents like styles and javascript.

## Deploy

First of all, deploy FlatPress.

Second, go to *`<my-flatpress-deployment-dir>`*`/site` and deploy this site with:

```bash
$ git clone https://github.com/diegolagoglez/flatpress-example-site
```

Finally, run `make` to build the site and access it with your favourite browser.

Oh, do not forget to configure your web server.
