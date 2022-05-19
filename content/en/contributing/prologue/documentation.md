---
title: "Documentation"
description: "Information on contributing documentation to the Authelia project."
lead: "Authelia has great documentation however there are always things that can be added. This section describes the contribution process for the documentation even though it's incredibly easy."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  contributing:
    parent: "prologue"
weight: 130
toc: true
---

## Introduction

The website is built on [Hugo] using the [Doks] theme. [Hugo] is a powerful website building tool which allows several
simple workflows for developers as well as numerous handy features like [Shortodes] which allow building reusable
parameterized sections of content.

## Making a Change

Anyone can simply edit the [Markdown] of the relevant document which shares a path with the website URL under the
[docs folder on GitHub]. In most if not all pages there is a link included at the very bottom which links directly to
the [Markdown] file responsible for the document.

## Viewing Changes

It's relatively easy to run the **Authelia** website locally to test out the changes you've made.

### Requirements

- [git] _(though this can be skipped if you just download the repository)_
- [Node.js] and npm _(bundled with [Node.js])_

### Directions

The following steps will allow you to run the website on the localhost and view it live in your browser:

1. Run the following commands:
  ```console
  $ git clone https://github.com/authelia/authelia.git
  $ cd authelia/docs
  $ npm run install
  $ npm run start
  ```
2. Visit [http://localhost:1313/](http://localhost:1313/) in your browser.
3. Modify pages to see the effects live in your browser.

[Hugo]: https://gohugo.io/
[Shortodes]: https://gohugo.io/content-management/shortcodes/
[Doks]: https://getdoks.org/
[Markdown]: https://www.markdownguide.org/
[git]: https://git-scm.com/
[Node.js]: https://nodejs.org/en/

[docs folder on GitHub]: https://github.com/authelia/authelia/tree/master/docs
