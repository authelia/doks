---
title: "NGINX Proxy Manager"
description: "An integration guide for Authelia and the NGINX Proxy Manager reverse proxy"
lead: "A guide on integrating Authelia with NGINX Proxy Manager."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "proxies"
weight: 260
toc: true
---

[NGINX Proxy Manager] is supported by **Authelia**. It's a [NGINX] proxy with a configuration UI.

## UNDER CONSTRUCTION

While this proxy is supported we don't have any specific documentation for it at the present time. Please see the
[NGINX integration documentation](nginx.md) for hints on how to configure this.

## Forwarded Header Trust

It's important to read the [Forwarded Headers] section as part of any proxy configuration.

To configure trusted proxies for [NGINX Proxy Manager] see the [NGINX] section on
[Forwarded Header Trust](nginx.md#forwarded-header-trust). Adapting this to [NGINX Proxy Manager] is beyond the scope of
this documentation.

## See Also

- [NGINX Proxy Manager Documentation](https://nginxproxymanager.com/setup/)
- [NGINX ngx_http_auth_request_module Module Documentation](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html)
- [Forwarded Headers]

[NGINX Proxy Manager]: https://nginxproxymanager.com/
[NGINX]: https://www.nginx.com/
[Forwarded Headers]: fowarded-headers
