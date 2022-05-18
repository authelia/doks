---
title: "Traefik"
description: "An integration guide for Authelia and the Traefik reverse proxy"
lead: "A guide on integrating Authelia with the Traefik reverse proxy."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "proxies"
weight: 280
toc: true
---

[Traefik] is a reverse proxy supported by **Authelia**.

## Requirements

You need the following to run **Authelia** with [Traefik]:

- [Traefik] v2.0.0 or higher (a guide exists for 1.x [here](traefikv1.md))
- [Traefik] v2.4.1 or higher if you wish to use [basic authentication](#basic-authentication)

## Forwarded Header Trust

It's important to read the [Forwarded Headers] section as part of any proxy configuration.

With [Traefik] this is controlled at the [Entry Point](https://doc.traefik.io/traefik/routing/entrypoints) level which
is the name they give the HTTP/HTTPS listener. The examples set the trusted range to
`127.0.0.1/32,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16` which is the [Traefik] container, and all private network ranges.

See the lines with `forwardedHeaders.trustedIPs` and `proxyProtocol.trustedIPs`. It's strongly recommended that you
adjust these values to your needs.

## Configuration

Below you will find commented examples of the following docker deployment:

* [Traefik]
* Authelia portal
* Protected endpoint (Nextcloud)
* Protected endpoint with [Authorization] header for basic authentication (Heimdall)

The below configuration looks to provide examples of running [Traefik] 2.x with labels to protect your endpoint
(Nextcloud in this case).

Please ensure that you also setup the respective [ACME configuration](https://docs.traefik.io/https/acme/) for your
[Traefik] setup as this is not covered in the example below.

##### Docker Compose

This is an example configuration using [docker compose] labels:

```yaml
version: '3.8'

networks:
  net:
    driver: bridge

services:
  traefik:
    image: traefik:v2.6
    container_name: traefik
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - net
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.api.rule=Host(`traefik.example.com`)'
      - 'traefik.http.routers.api.entryPoints=https'
      - 'traefik.http.routers.api.service=api@internal'
      - 'traefik.http.routers.api.middlewares=authelia@docker'
      - 'traefik.http.routers.api.tls=true'
    ports:
      - "80:80"
      - "443:443"
    command:
      - '--api'
      - '--providers.docker=true'
      - '--providers.docker.exposedByDefault=false'
      - '--entryPoints.http=true'
      - '--entryPoints.http.address=:80'
      - '--entryPoints.http.forwardedHeaders.trustedIPs=127.0.0.1/32,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16'
      - '--entryPoints.http.proxyProtocol.trustedIPs=127.0.0.1/32,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16'
      - '--entryPoints.http.http.redirections.entryPoint.to=https'
      - '--entryPoints.http.http.redirections.entryPoint.scheme=https'
      - '--entryPoints.https=true'
      - '--entryPoints.https.address=:443'
      - '--entryPoints.https.forwardedHeaders.trustedIPs=127.0.0.1/32,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16'
      - '--entryPoints.https.proxyProtocol.trustedIPs=127.0.0.1/32,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16'
      - '--log=true'
      - '--log.level=DEBUG'
      - '--log.filepath=/var/log/traefik.log'

  authelia:
    image: authelia/authelia
    container_name: authelia
    volumes:
      - /path/to/authelia:/config
    networks:
      - net
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.authelia.rule=Host(`auth.example.com`)'
      - 'traefik.http.routers.authelia.entryPoints=https'
      - 'traefik.http.routers.authelia.tls=true'
      - 'traefik.http.middlewares.authelia.forwardAuth.address=http://authelia:9091/api/verify?rd=https://auth.example.com/'
      - 'traefik.http.middlewares.authelia.forwardAuth.trustForwardHeader=true'
      - 'traefik.http.middlewares.authelia.forwardAuth.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email'
      - 'traefik.http.middlewares.authelia-basic.forwardAuth.address=http://authelia:9091/api/verify?auth=basic'
      - 'traefik.http.middlewares.authelia-basic.forwardAuth.trustForwardHeader=true'
      - 'traefik.http.middlewares.authelia-basic.forwardAuth.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email'
    expose:
      - 9091
    restart: unless-stopped
    environment:
      TZ: "Australia/Melbourne"

  nextcloud:
    image: linuxserver/nextcloud
    container_name: nextcloud
    volumes:
      - /path/to/nextcloud/config:/config
      - /path/to/nextcloud/data:/data
    networks:
      - net
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.nextcloud.rule=Host(`nextcloud.example.com`)'
      - 'traefik.http.routers.nextcloud.entryPoints=https'
      - 'traefik.http.routers.nextcloud.tls=true'
      - 'traefik.http.routers.nextcloud.middlewares=authelia@docker'
    expose:
      - 443
    restart: unless-stopped
    environment:
      PUID: "1000"
      PGID: "1000"
      TZ: "Australia/Melbourne"

  heimdall:
    image: linuxserver/heimdall
    container_name: heimdall
    volumes:
      - /path/to/heimdall/config:/config
    networks:
      - net
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.heimdall.rule=Host(`heimdall.example.com`)'
      - 'traefik.http.routers.heimdall.entryPoints=https'
      - 'traefik.http.routers.heimdall.tls=true'
      - 'traefik.http.routers.heimdall.middlewares=authelia-basic@docker'
    expose:
      - 443
    restart: unless-stopped
    environment:
      PUID: "1000"
      PGID: "1000"
      TZ: "Australia/Melbourne"
```

## FAQ

### Basic Authentication

Authelia provides the means to be able to authenticate your first factor via the [Proxy-Authorization] header, this
is compatible with [Traefik].

If you have a use-case which requires the use of the [Authorization] header/basic authentication login prompt you can
call Authelia's `/api/verify?auth=basic` endpoint to force a switch to the [Authorization] header.

### Middleware authelia@docker not found

If [Traefik] and **Authelia** are defined in different docker compose stacks you may experience an issue where [Traefik]
complains that: `middleware authelia@docker not found`.

This can be avoided a couple different ways:

1. Ensure **Authelia** container is up before [Traefik] is started:
  - Utilise the [depends_on](https://docs.docker.com/compose/compose-file/#depends_on) option
2. Define the **Authelia** middleware on your [Traefik] container:

```yaml
- 'traefik.http.middlewares.authelia.forwardAuth.address=http://authelia:9091/api/verify?rd=https://auth.example.com/'
- 'traefik.http.middlewares.authelia.forwardAuth.trustForwardHeader=true'
- 'traefik.http.middlewares.authelia.forwardAuth.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email'
```

## See Also

- [Traefik ForwardAuth Documentation](https://doc.traefik.io/traefik/middlewares/http/forwardauth/)
- [Traefik Forwarded Headers Documentation](https://doc.traefik.io/traefik/routing/entrypoints/#forwarded-headers)
- [Traefik Proxy Protocol Documentation](https://doc.traefik.io/traefik/routing/entrypoints/#proxyprotocol)
- [Forwarded Headers]

[docker compose]: https://docs.docker.com/compose/
[Traefik]: https://docs.traefik.io/
[Forwarded Headers]: fowarded-headers
[Authorization]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization
[Proxy-Authorization]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authorization
