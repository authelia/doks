---
title: "Traefik v1"
description: "An integration guide for Authelia and the Traefik v1 reverse proxy"
lead: "A guide on integrating Authelia with the Traefik reverse proxy."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "proxies"
weight: 290
toc: true
---

[Traefik] v1 is a reverse proxy supported by **Authelia**.

_**Important:** When using these guides it's important to recognize that we cannot provide a guide for every possible
method of deploying a proxy. These are guides showing a suggested setup only and you need to understand the proxy
configuration and customize it to your needs. To-that-end we include links to the official proxy documentation
throughout this documentation and in the [See Also](#see-also) section._

## Trusted Proxies

_**Important:** You should read the [Forwarded Headers] section and this section as part of any proxy configuration.
Especially if you have never read it before._

_**Important:** The included example is **NOT** meant for production use. It's used expressly as an example to showcase
how you can configure multiple IP ranges. You should customize this example to fit your specific architecture and needs.
You should only include the specific IP address ranges of the trusted proxies within your architecture and should not
trust entire subnets unless that subnet only has trusted proxies and no other services._

## Configuration

Below you will find commented examples of the following docker deployment:

* [Traefik] 1.x
* Authelia portal
* Protected endpoint (Nextcloud)
* Protected endpoint with `Authorization` header for basic authentication (Heimdall)

The below configuration looks to provide examples of running [Traefik] v1 with labels to protect your endpoint
(Nextcloud in this case).

Please ensure that you also setup the respective [ACME](https://docs.traefik.io/v1.7/configuration/acme/) configuration
for your [Traefik] setup as this is not covered in the example below.

### Basic Authentication

Authelia provides the means to be able to authenticate your first factor via the `Proxy-Authorization` header.
Given that this is not compatible with [Traefik] 1.x you can call the **Authelia** `/api/verify` endpoint with the
`auth=basic` query parameter to force a switch to the `Authentication` header.

##### docker-compose.yml
```yml
version: '3'

networks:
  net:
    driver: bridge
services:
  traefik:
    image: traefik:v1.7.34-alpine
    container_name: traefik
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - net
    labels:
      - 'traefik.frontend.rule=Host:traefik.example.com'
      - 'traefik.port=8081'
    ports:
      - 80:80
      - 443:443
      - 8081:8081
    restart: unless-stopped
    command:
      - '--api'
      - '--api.entrypoint=api'
      - '--docker'
      - '--defaultEntryPoints=https'
      - '--logLevel=DEBUG'
      - '--traefikLog=true'
      - '--traefikLog.filepath=/var/log/traefik.log'
      - '--entryPoints=Name:http Address::80'
      - '--entryPoints=Name:https Address::443 TLS'
      ## See the Forwarded Header Trust section. Comment the above two lines, then uncomment and customize the next two lines to configure the TrustedIPs.
      # - '--entryPoints=Name:http Address::80 ForwardedHeaders.TrustedIPs:10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7 ProxyProtocol.TrustedIPs:10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7'
      # - '--entryPoints=Name:https Address::443 TLS ForwardedHeaders.TrustedIPs:10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7 ProxyProtocol.TrustedIPs:10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7'
      - '--entryPoints=Name:api Address::8081'
  authelia:
    image: authelia/authelia
    container_name: authelia
    volumes:
      - /path/to/authelia:/config
    networks:
      - net
    labels:
      - 'traefik.frontend.rule=Host:login.example.com'
    expose:
      - 9091
    restart: unless-stopped
    environment:
      - TZ=Australia/Melbourne
  nextcloud:
    image: linuxserver/nextcloud
    container_name: nextcloud
    volumes:
      - /path/to/nextcloud/config:/config
      - /path/to/nextcloud/data:/data
    networks:
      - net
    labels:
      - 'traefik.frontend.rule=Host:nextcloud.example.com'
      - 'traefik.frontend.auth.forward.address=http://authelia:9091/api/verify?rd=https://login.example.com/'
      - 'traefik.frontend.auth.forward.trustForwardHeader=true'
      - 'traefik.frontend.auth.forward.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email'
    expose:
      - 443
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Australia/Melbourne
  heimdall:
    image: linuxserver/heimdall
    container_name: heimdall
    volumes:
      - /path/to/heimdall/config:/config
    networks:
      - net
    labels:
      - 'traefik.frontend.rule=Host:heimdall.example.com'
      - 'traefik.frontend.auth.forward.address=http://authelia:9091/api/verify?auth=basic'
      - 'traefik.frontend.auth.forward.trustForwardHeader=true'
      - 'traefik.frontend.auth.forward.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email'
    expose:
      - 443
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Australia/Melbourne
```

## See Also

- [Traefik v1 Documentation](https://doc.traefik.io/traefik/v1.7/)
- [Forwarded Headers]

[Traefik]: https://docs.traefik.io/v1.7/
[Forwarded Headers]: fowarded-headers
