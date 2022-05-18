---
title: "Envoy"
description: "An integration guide for Authelia and the Envoy reverse proxy"
lead: "A guide on integrating Authelia with the Envoy reverse proxy."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "proxies"
weight: 230
toc: true
---

[Envoy] is probably supported by **Authelia**.

## UNDER CONSTRUCTION

It's currently not certain, but fairly likely that [Envoy] is supported by **Authelia**. We wish to add documentation
and thus if anyone has this working please let us know.

We will aim to perform documentation for this on our own but there is no current timeframe.

## Forwarded Header Trust

It's important to read the [Forwarded Headers] section as part of any proxy configuration.

## Potential

Support for [Envoy] should be possible via [Envoy]'s
[external authorization](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/ext_authz/v3/ext_authz.proto.html#extensions-filters-http-ext-authz-v3-extauthz).

## See Also

- [Envoy External Authorization Documentation](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/ext_authz/v3/ext_authz.proto.html#extensions-filters-http-ext-authz-v3-extauthz)
- [Forwarded Headers]

[Envoy]: https://www.envoyproxy.io/
[Forwarded Headers]: fowarded-headers
