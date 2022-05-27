---
title: "Bare-Metal"
description: "Deploying Authelia on Bare-Metal."
lead: "Authelia can be deployed on Bare-Metal as long as it sits behind a proxy."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "deployment"
weight: 230
toc: true
---

There are several ways to achieve this, as _Authelia_ runs as a daemon. We do not provide specific examples for running
_Authelia_ as a service excluding the [systemd unit](#systemd) files.

## systemd

We publish two example [systemd] unit files:

- [authelia.service](https://github.com/authelia/authelia/blob/master/authelia.service)
- [authelia@.service](https://github.com/authelia/authelia/blob/master/authelia%40.service)

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
