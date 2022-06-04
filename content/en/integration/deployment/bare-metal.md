---
title: "Bare-Metal"
description: "Deploying Authelia on Bare-Metal."
lead: "Authelia can be deployed on Bare-Metal as long as it sits behind a proxy."
date: 2022-05-27T22:24:38+10:00
lastmod: 2022-06-03T10:43:55+10:00
draft: false
images: []
menu:
  integration:
    parent: "deployment"
weight: 250
toc: true
---

There are several ways to achieve this, as *Authelia* runs as a daemon. We do not provide specific examples for running
*Authelia* as a service excluding the [systemd unit](#systemd) files.

## systemd

We publish two example [systemd] unit files:

* [authelia.service](https://github.com/authelia/authelia/blob/master/authelia.service)
* [authelia@.service](https://github.com/authelia/authelia/blob/master/authelia%40.service)

## Arch Linux

In addition to the [binaries](#binaries) we publish, we also publish an
[AUR Package](https://aur.archlinux.org/packages/authelia).

## Debian

We publish `.deb` packages with our [releases] which can be installed
on most Debian based operating systems.

### APT Repository

In addition to the `.deb` packages we also have an [APT Repository](https://apt.authelia.com).

## FreeBSD

In addition to the [binaries](#binaries) we publish, [FreshPorts](https://www.freshports.org/www/authelia/) offer a
package.

## Binaries

We publish binaries with our [releases] which can be installed on many operating systems.

[releases]: https://github.com/authelia/authelia/releases
[systemd]: https://systemd.io/
