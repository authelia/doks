---
title: "Authentication"
description: "An overview of a authentication."
lead: "An overview of a authentication."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  overview:
    parent: "prologue"
weight: 210
toc: false
---

Multi-Factor Authentication or MFA as a concept is separated into three major categories. These categories are:

- something you know
- something you have
- something you are

Modern best security practice dictates that using multiple of these categories is the necessary for security. Users are
unreliable and simple usernames and passwords are not sufficient for security.

**Authelia** enables primarily two-factor authentication. These methods offered come in two forms:

- 1FA or first-factor authentication which is handled by a username and password. This falls into the _something you know_
  categorization.
- 2FA or second-factor authentication which is handled by several methods including one-time passwords, authentication
  keys, etc. This falls into the _something you have_ categorization.

In addition to this Authelia can apply authorization policies to individual website resources which restrict which
identities can access which resources from a given remote address. These policies can require 1FA, 2FA, or outright deny
access depending on the criteria you configure.

