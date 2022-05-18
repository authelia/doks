---
title: "Caddy"
description: "An integration guide for Authelia and the Caddy reverse proxy"
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

**Authelia** offers integration support for the official forward auth integration method Caddy provides, we don't
officially support any plugin that supports this though we don't specifically prevent such plugins working and there may
be plugins that work fine provided they support the forward authentication specification correctly.

_**Important:** When using these guides it's important to recognize that we cannot provide a guide for every possible
method of deploying a proxy. These are guides showing a suggested setup only and you need to understand the proxy
configuration and customize it to your needs. To-that-end we include links to the official proxy documentation
throughout this documentation and in the [See Also](#see-also) section._

## Requirements

You need the following to run **Authelia** with [Caddy]:

- [Caddy] [v2.5.1](https://github.com/caddyserver/caddy/releases/tag/v2.5.1) or greater
You must be running [Caddy] [2.5.1](https://github.com/caddyserver/caddy/releases/tag/v2.5.1) or greater for this
configuration to work.

## Forwarded Header Trust

It's important to read the [Forwarded Headers] section as part of any proxy configuration.

[Caddy] by default strips these headers entirely which is a good security practice. To preserve these headers from
trusted proxies you need to uncomment and configure the `trusted_proxies` directive in the `(trusted_proxy_list)` at the
top of the examples with a list of IP ranges that you consider to be trustworthy.

The example `(trusted_proxy_list)` [Caddy Snippet] is not meant for production use and is an example of adding multiple
network subnets to be considered trustworthy. You should read the [Caddy Trusted Proxies Documentation] as part of
configuring this. It's important to ensure you take the time to configure this carefully and correctly.

## Configuration

Below you will find commented examples of the following configuration:

- Authelia Portal
- Protected Endpoint (Nextcloud)

### Basic examples

This example is the preferred example for integration with [Caddy]. There is an [advanced example](#advanced-example)
but we _**strongly urge**_ anyone who needs to use this for a particular reason to either reach out to us or Caddy for
support to ensure the basic example covers your use case in a secure way.

#### Subdomain

```caddyfile
## This snippet is used to configure the list of trusted proxies as per the Forwarded Header Trust docs section.
## It's recommended that both the Forwarded Header Section, and the official Caddy
## Trusted Proxies Documentation (which is linked in the See Also section of this document) is read before deciding
## how to configure this. This snippet needs to be imported in all forward_auth and reverse_proxy sections protected
## by Authelia.
(trusted_proxy_list) {
       ## Uncomment & adjust the following line to configure specific ranges which should be considered as trustworthy.
       # trusted_proxies 10.0.0.0/8 172.16.0.0/16 192.168.0.0/16 fc00::/7
}

# Authelia Portal.
auth.example.com {
        reverse_proxy authelia:9091 {
                ## This import needs to be included if you're relying on a trusted proxies configuration.
                import trusted_proxy_list
        }
}

# Protected Endpoint.
nextcloud.example.com {
        forward_auth authelia:9091 {
                uri /api/verify?rd=https://auth.example.com
                copy_headers Remote-User Remote-Groups Remote-Name Remote-Email

                ## This import needs to be included if you're relying on a trusted proxies configuration.
                import trusted_proxy_list
        }
        reverse_proxy nextcloud:80 {
                ## This import needs to be included if you're relying on a trusted proxies configuration.
                import trusted_proxy_list
        }
}
```

#### Subpath

```caddyfile
## This snippet is used to configure the list of trusted proxies as per the Forwarded Header Trust docs section.
## It's recommended that both the Forwarded Header Section, and the official Caddy
## Trusted Proxies Documentation (which is linked in the See Also section of this document) is read before deciding
## how to configure this. This snippet needs to be imported in all forward_auth and reverse_proxy sections protected
## by Authelia.
(trusted_proxy_list) {
       ## Uncomment & adjust the following line to configure specific ranges which should be considered as trustworthy.
       # trusted_proxies 10.0.0.0/8 172.16.0.0/16 192.168.0.0/16 fc00::/7
}

example.com {
        # Authelia Portal.
        @authelia path /authelia /authelia/*
        handle @authelia {
                reverse_proxy authelia:9091 {
                        ## This import needs to be included if you're relying on a trusted proxies configuration.
                        import trusted_proxy_list
                }
        }

        # Protected Endpoint.
        @nextcloud path /nextcloud /nextcloud/*
        handle @nextcloud {
                forward_auth authelia:9091 {
                        uri /api/verify?rd=https://example.com/authelia
                        copy_headers Remote-User Remote-Groups Remote-Name Remote-Email

                        ## This import needs to be included if you're relying on a trusted proxies configuration.
                        import trusted_proxy_list
                }
                reverse_proxy nextcloud:80 {
                        ## This import needs to be included if you're relying on a trusted proxies configuration.
                        import trusted_proxy_list
                }
        }
}
```

### Advanced example

The advanced example allows for more flexible customization, however the [basic example](#basic-examples) should be
preferred in _most_ situations. If you are unsure of what you're doing please don't use this method.

_**Important:** Making a mistake when configuring the advanced example could lead to authentication bypass or errors._

```caddyfile
## This snippet is used to configure the list of trusted proxies as per the Forwarded Header Trust docs section.
## It's recommended that both the Forwarded Header Section, and the official Caddy
## Trusted Proxies Documentation (which is linked in the See Also section of this document) is read before deciding
## how to configure this. This snippet needs to be imported in all forward_auth and reverse_proxy sections protected
## by Authelia.
(trusted_proxy_list) {
       ## Uncomment & adjust the following line to configure specific ranges which should be considered as trustworthy.
       # trusted_proxies 10.0.0.0/8 172.16.0.0/16 192.168.0.0/16 fc00::/7
}

# Authelia Portal.
auth.example.com {
        reverse_proxy authelia:9091 {
                ## This import needs to be included if you're relying on a trusted proxies configuration.
                import trusted_proxy_list
        }
}

# Protected Endpoint.
nextcloud.example.com {
        route {
                reverse_proxy authelia:9091 {
                        ## This import needs to be included if you're relying on a trusted proxies configuration.
                        import trusted_proxy_list

                        method GET
                        rewrite "/api/verify?rd=https://auth.example.com"

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

                reverse_proxy nextcloud:80 {
                        ## This import needs to be included if you're relying on a trusted proxies configuration.
                        import trusted_proxy_list
                }
        }
}
```

## See Also

- [Caddy General Documentation](https://caddyserver.com/docs/)
- [Caddy Forward Auth Documentation]
- [Caddy Trusted Proxies Documentation]
- [Caddy Snippet] Documentation
- [Forwarded Headers]

[Caddy]: https://caddyserver.com
[Caddy Snippet]: https://caddyserver.com/docs/caddyfile/concepts#snippets
[Caddy Forward Auth Documentation]: https://caddyserver.com/docs/caddyfile/directives/forward_auth
[Caddy Trusted Proxies Documentation]: https://caddyserver.com/docs/caddyfile/directives/reverse_proxy#trusted_proxies
[Forwarded Headers]: fowarded-headers
