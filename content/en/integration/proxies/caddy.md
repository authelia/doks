---
title: "Caddy"
description: "Caddy Integration"
lead: "A guide on integrating Authelia with the Caddy reverse proxy."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "proxies"
weight: 220
toc: true
---

[Caddy] is a reverse proxy supported by **Authelia**.

_**Important:** [Caddy] officially supports the forward auth flow in version 2.5.1 and greater. You must be using this
version in order to use either Caddyfile._

Authelia offers integration support for the official forward auth integration method Caddy provides, we
can't reasonably be expected to offer support for all of the different plugins that exist.

## Requirements

You must be running [Caddy] [2.5.1](https://github.com/caddyserver/caddy/releases/tag/v2.5.1) or greater for this
configuration to work.

## Configuration

Below you will find commented examples of the following configuration:

* Authelia portal
* Protected endpoint (Nextcloud)

### Basic examples

This example is the preferred example for integration with [Caddy]. There is an [advanced example](#advanced-example) but
we _**strongly urge**_ anyone who needs to use this for a particular reason to either reach out to us or Caddy for support
to ensure the basic example covers your use case in a secure way.


#### Subdomain

```Caddyfile
authelia.example.com {
       reverse_proxy authelia:9091
}

nextcloud.example.com {
       forward_auth authelia:9091 {
               uri /api/verify?rd=https://authelia.example.com
               copy_headers Remote-User Remote-Groups Remote-Name Remote-Email
       }
       reverse_proxy nextcloud:80
}
```

#### Subpath

```Caddyfile
example.com {
       @authelia path /authelia /authelia/*
       handle @authelia {
               reverse_proxy authelia:9091
       }

       @nextcloud path /nextcloud /nextcloud/*
       handle @nextcloud {
               forward_auth authelia:9091 {
                       uri /api/verify?rd=https://example.com/authelia
                       copy_headers Remote-User Remote-Groups Remote-Name Remote-Email
               }
               reverse_proxy nextcloud:80
       }
}
```

## Advanced example

The advanced example allows for more flexible customization, however the [basic example](#basic-example) should be
preferred in _most_ situations. If you are unsure of what you're doing please don't use this method.

_**Important:** Making a mistake when configuring the advanced example could lead to authentication bypass or errors._

```Caddyfile
authelia.example.com {
       reverse_proxy authelia:9091
}

nextcloud.example.com {
       route {
               reverse_proxy authelia:9091 {
                       method GET
                       rewrite "/api/verify?rd=https://authelia.example.com"

                       header_up X-Forwarded-Method {method}
                       header_up X-Forwarded-Uri {uri}

                       ## If the auth request:
                       ##   1. Responds with a status code IN the 200-299 range.
                       ## Then:
                       ##   1. Proxy the request to the backend.
                       ##   2. Copy the relevant headers from the auth request and provide them to the backend.
                       @good status 2xx
                       handle_response @good {
                               request_header {
                                       Remote-User {http.reverse_proxy.header.Remote-User}
                                       Remote-Groups {http.reverse_proxy.header.Remote-Groups}
                                       Remote-Name {http.reverse_proxy.header.Remote-Name}
                                       Remote-Email {http.reverse_proxy.header.Remote-Email}
                               }
                       }

                       ## If the auth request:
                       ##   1. Responds with a status code NOT IN the 200-299 range.
                       ## Then:
                       ##   1. Respond with the status code of the auth request.
                       ##   1. Copy the response except for several headers.
                       @denied {
                               status 1xx 3xx 4xx 5xx
                       }
                       handle_response @denied {
                               copy_response
                               copy_response_headers {
                                       exclude Connection Keep-Alive Te Trailers Transfer-Encoding Upgrade
                               }
                       }
               }

               reverse_proxy nextcloud:80
       }
}
```

[Caddy]: https://caddyserver.com
