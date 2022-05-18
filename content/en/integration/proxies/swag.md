---
title: "SWAG"
description: "An integration guide for Authelia and the SWAG reverse proxy"
lead: "A guide on integrating Authelia with SWAG."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "proxies"
weight: 270
toc: true
---

[SWAG] is a reverse proxy supported by **Authelia**. It's an [NGINX] proxy container with bundled configurations to make
your life easier.

## Introduction

As [SWAG] is a [NGINX] proxy with curated configurations, integration of **Authelia** with [SWAG] is very easy and you
only need to enabled two includes.

_**Note:** All paths in this guide are the locations inside the container. You will have to either edit the files within
the container or adapt the path to the path you have mounted the relevant container path to._

## Forwarded Header Trust

It's important to read the [Forwarded Headers] section as part of any proxy configuration.

To configure trusted proxies for [SWAG] see the [NGINX] section on
[Forwarded Header Trust](nginx.md#forwarded-header-trust). Adapting this to [SWAG] is beyond the scope of this
documentation.

## Prerequisite Steps

These steps must be followed regardless of the choice of [subdomain](#subdomain-steps) or [subpath](#subpath-steps).

1. Deploy **Authelia** to your docker network with the `container_name` of `authelia` and ensure it's listening on the
   default port and you have not configured the **Authelia** server TLS settings.

## Subdomain Steps

In the server configuration for the application you want to protect:

1. Edit the `/config/nginx/proxy-confs/` file for the application you wish to protect.
2. Uncomment the `#include /config/nginx/authelia-server.conf;` line which should be within the `server` block
   but not inside any `location` blocks.
3. Uncomment the `#include /config/nginx/authelia-location.conf;` line which should be within the applications
   `location` block.

### Example

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name heimdall.*;

    include /config/nginx/ssl.conf;

    client_max_body_size 0;

    # Authelia: Step 1.
    include /config/nginx/authelia-server.conf;

    location / {
        # Authelia: Step 2.
        include /config/nginx/authelia-location.conf;

        include /config/nginx/proxy.conf;
        resolver 127.0.0.11 valid=30s;
        set $upstream_app heimdall;
        set $upstream_port 443;
        set $upstream_proto https;
        proxy_pass $upstream_proto://$upstream_app:$upstream_port;
    }
}
```

## Subpath Steps

_**Note:** Steps 1 and 2 only need to be done once, even if you wish to protect multiple applications._

1. Edit `/config/nginx/proxy-confs/default`.
2. Uncomment the `#include /config/nginx/authelia-server.conf;` line.
3. Edit the `/config/nginx/proxy-confs/` file for the application you wish to protect.
4. Uncomment the `#include /config/nginx/authelia-location.conf;` line which should be within the applications
   `location` block.

### Example

```nginx
location ^~ /bazarr/ {
    # Authelia: Step 4.
    include /config/nginx/authelia-location.conf;

    include /config/nginx/proxy.conf;
    resolver 127.0.0.11 valid=30s;
    set $upstream_app bazarr;
    set $upstream_port 6767;
    set $upstream_proto http;
    proxy_pass $upstream_proto://$upstream_app:$upstream_port;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
}
```

## See Also

- [Authelia NGINX Integration Documentation](nginx.md)
- [LinuxServer.io Setting Up Authelia With SWAG Documentation / Blog Post](https://www.linuxserver.io/blog/2020-08-26-setting-up-authelia)
- [NGINX ngx_http_auth_request_module Module Documentation](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html)
- [Forwarded Headers]

[SWAG]: https://docs.linuxserver.io/general/swag
[NGINX]: https://www.nginx.com/
[Forwarded Headers]: fowarded-headers
