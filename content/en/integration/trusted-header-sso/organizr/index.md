---
title: "Organizr"
description: "Trusted Header SSO Integration for Organizr"
lead: "This documentation is maintained by the community. This documentation is not guaranteed to be complete or up-to-date. If you find an error with this documentation please either make a pull request or start a GitHub Discussion."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "trusted-header-sso"
weight: 620
toc: true
---

This is a guide on integration of **Authelia** and [Organizr] via the trusted header SSO authentication.

As with all guides in this section it's important you read the [introduction](../introduction.md) first.

## Tested Versions

- Authelia: v4.35.5
- Organizr: 2.1.1890

## Before You Begin

This example makes the following assumptions:

- **Application Root URL:** `https://organizr.example.com`
- **Authelia Root URL:** `https://auth.example.com`
- **Reverse Proxy IP:** `172.16.0.1`

## Configuration

To configure [Organizr] to trust the `Remote-User` and `Remote-Email` header do the following:

1. Visit System Settings
2. Visit Main
3. Visit Auth Proxy
4. Fill in the following information:
   1. Auth Proxy: `Enabled`
   2. Auth Proxy Whitelist: `172.16.0.1`
   3. Auth Proxy Header Name: `Remote-User`
   4. Auth Proxy Header Name for Email: `Remote-Email`
   5. Override Logout: `Enabled`
   6. Logout URL: `https://auth.example.com/logout`


{{< figure src="organizr.png" alt="Organizr" width="736" style="padding-right: 10px" >}}

[Organizr]: https://organizr.app/
