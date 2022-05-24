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

_**Important:** When using these guides it's important to recognize that we cannot provide a guide for every possible
method of deploying a proxy. These are guides showing a suggested setup only and you need to understand the proxy
configuration and customize it to your needs. To-that-end we include links to the official proxy documentation
throughout this documentation and in the [See Also](#see-also) section._

## Requirements

You need the following to run **Authelia** with [Traefik]:

- [Traefik] [v2.0.0](https://github.com/traefik/traefik/releases/tag/v2.0.0) or greater
  (a guide exists for 1.x [here](traefikv1.md))
- [Traefik] [v2.4.1](https://github.com/traefik/traefik/releases/tag/v2.4.1) or greater if you wish to use
  [basic authentication](#basic-authentication)

## Trusted Proxies

_**Important:** You should read the [Forwarded Headers] section and this section as part of any proxy configuration.
Especially if you have never read it before._

_**Important:** The included example is **NOT** meant for production use. It's used expressly as an example to showcase
how you can configure multiple IP ranges. You should customize this example to fit your specific architecture and needs.
You should only include the specific IP address ranges of the trusted proxies within your architecture and should not
trust entire subnets unless that subnet only has trusted proxies and no other services._

With [Traefik] this is controlled at the [Entry Point](https://doc.traefik.io/traefik/routing/entrypoints) level which
is the name they give the HTTP/HTTPS listener. The examples set the trusted range to `192.168.253.20/32` and
`192.168.253.21/32` which are two arbitrary hosts. It's expected you'll uncomment and configure these values.

[Traefik] requires users implicitly configure which proxies are considered trusted. To configure a proxy as trusted
please see the lines with `trustedIPs`. It's strongly recommended that you adjust these values to your needs.

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

### Docker Compose

This is an example configuration using [docker compose] labels:

##### docker-compose.yml

```yaml
version: "3.8"
networks:
  net:
    driver: bridge
services:
  traefik:
    container_name: traefik
    image: traefik:v2.6
    restart: unless-stopped
    command:
      - '--api=true'
      - '--api.dashboard=true'
      - '--api.insecure=false'
      - '--pilot.dashboard=false'
      - '--global.sendAnonymousUsage=false'
      - '--global.checkNewVersion=false'
      - '--log=true'
      - '--log.level=DEBUG'
      - '--log.filepath=/config/traefik.log'
      - '--providers.docker=true'
      - '--providers.docker.exposedByDefault=false'
      - '--entryPoints.http=true'
      - '--entryPoints.http.address=:8080/tcp'
      - '--entryPoints.http.http.redirections.entryPoint.to=https'
      - '--entryPoints.http.http.redirections.entryPoint.scheme=https'
      ## Please see the Forwarded Header Trust section of the Authelia Traefik Integration documentation.
      # - '--entryPoints.http.forwardedHeaders.trustedIPs=10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7'
      # - '--entryPoints.http.proxyProtocol.trustedIPs=10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7'
      - '--entryPoints.http.forwardedHeaders.insecure=false'
      - '--entryPoints.http.proxyProtocol.insecure=false'
      - '--entryPoints.https=true'
      - '--entryPoints.https.address=:8443/tcp'
      ## Please see the Forwarded Header Trust section of the Authelia Traefik Integration documentation.
      # - '--entryPoints.https.forwardedHeaders.trustedIPs=10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7'
      # - '--entryPoints.https.proxyProtocol.trustedIPs=10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,fc00::/7'
      - '--entryPoints.https.forwardedHeaders.insecure=false'
      - '--entryPoints.https.proxyProtocol.insecure=false'
    networks:
      net: {}
    ports:
      - "80:8080"
      - "443:8443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ${PWD}/data/traefik:/config
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.api.rule=Host(`traefik.example.com`)'
      - 'traefik.http.routers.api.entryPoints=https'
      - 'traefik.http.routers.api.tls=true'
      - 'traefik.http.routers.api.service=api@internal'
      - 'traefik.http.routers.api.middlewares=authelia@docker'
  authelia:
    container_name: authelia
    image: authelia/authelia
    restart: unless-stopped
    networks:
      net: {}
    expose:
      - 9091
    volumes:
      - ${PWD}/data/authelia/config:/config
    environment:
      TZ: "Australia/Melbourne"
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
  nextcloud:
    container_name: nextcloud
    image: linuxserver/nextcloud
    restart: unless-stopped
    networks:
      net: {}
    expose:
      - 443
    volumes:
      - ${PWD}/data/nextcloud/config:/config
      - ${PWD}/data/nextcloud/data:/data
    environment:
      PUID: "1000"
      PGID: "1000"
      TZ: "Australia/Melbourne"
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.nextcloud.rule=Host(`nextcloud.example.com`)'
      - 'traefik.http.routers.nextcloud.entryPoints=https'
      - 'traefik.http.routers.nextcloud.tls=true'
      - 'traefik.http.routers.nextcloud.middlewares=authelia@docker'
  heimdall:
    container_name: heimdall
    image: linuxserver/heimdall
    restart: unless-stopped
    networks:
      net: {}
    expose:
      - 443
    volumes:
      - ${PWD}/data/heimdall/config:/config
    environment:
      PUID: "1000"
      PGID: "1000"
      TZ: "Australia/Melbourne"
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.heimdall.rule=Host(`heimdall.example.com`)'
      - 'traefik.http.routers.heimdall.entryPoints=https'
      - 'traefik.http.routers.heimdall.tls=true'
      - 'traefik.http.routers.heimdall.middlewares=authelia-basic@docker'
```

### YAML

This example uses a `docker-compose.yml` similar to the one above however it has two major differences:

1. A majority of the configuration is in YAML instead of the `labels` section of the `docker-compose.yml` file.
2. It connects to **Authelia** over TLS with client CA certificates which ensures that [Traefik] is a proxy
   authorized to communicate with **Authelia**. This expects that the
   [Server TLS](../../configuration/miscellaneous/server.md#tls) section is configured correctly.

##### docker-compose.yml

```yaml
version: "3.8"
networks:
  net:
    driver: bridge
services:
  traefik:
    container_name: traefik
    image: traefik:v2.6
    restart: unless-stopped
    command:
      - '--api=true'
      - '--api.dashboard=true'
      - '--api.insecure=false'
      - '--pilot.dashboard=false'
      - '--global.sendAnonymousUsage=false'
      - '--global.checkNewVersion=false'
      - '--log=true'
      - '--log.level=DEBUG'
      - '--log.filepath=/config/traefik.log'
      - '--providers.docker=true'
      - '--providers.docker.exposedByDefault=false'
      - '--providers.file=true'
      - '--providers.file.watch=true'
      - '--providers.file.directory=/config/dynamic'
      - '--entryPoints.http=true'
      - '--entryPoints.http.address=:8080/tcp'
      - '--entryPoints.http.http.redirections.entryPoint.to=https'
      - '--entryPoints.http.http.redirections.entryPoint.scheme=https'
      - '--entryPoints.https=true'
      - '--entryPoints.https.address=:8443/tcp'
    networks:
      net: {}
    ports:
      - "80:8080"
      - "443:8443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ${PWD}/data/traefik/config:/config
      - ${PWD}/data/traefik/certificates:/certificates
    labels:
      - 'traefik.enable=true'
  authelia:
    container_name: authelia
    image: authelia/authelia
    restart: unless-stopped
    networks:
      net: {}
    expose:
      - 9091
    volumes:
      - ${PWD}/data/authelia/config:/config
      - ${PWD}/data/authelia/certificates:/certificates
    environment:
      TZ: "Australia/Melbourne"
    labels:
      - 'traefik.enable=true'
  nextcloud:
    container_name: nextcloud
    image: linuxserver/nextcloud
    restart: unless-stopped
    networks:
      net: {}
    expose:
      - 443
    volumes:
      - ${PWD}/data/nextcloud/config:/config
      - ${PWD}/data/nextcloud/data:/data
    environment:
      PUID: "1000"
      PGID: "1000"
      TZ: "Australia/Melbourne"
    labels:
      - 'traefik.enable=true'
  heimdall:
    container_name: heimdall
    image: linuxserver/heimdall
    restart: unless-stopped
    networks:
      net: {}
    expose:
      - 443
    volumes:
      - ${PWD}/data/heimdall/config:/config
    environment:
      PUID: "1000"
      PGID: "1000"
      TZ: "Australia/Melbourne"
    labels:
      - 'traefik.enable=true'
  whoami:
    container_name: whoami
    image: traefik/whoami:latest
    restart: unless-stopped
    networks:
      net: {}
    expose:
      - 80
    environment:
      TZ: "Australia/Melbourne"
    labels:
      - "traefik.enable=true"

```

##### traefik.yml

```yaml
entryPoints:
  web:
    proxyProtocol:
      insecure: false
      trustedIPs: []
    forwardedHeaders:
      insecure: false
      trustedIPs: []
  websecure:
    proxyProtocol:
      insecure: false
      trustedIPs: []
    forwardedHeaders:
      insecure: false
      trustedIPs: []
tls:
  options:
    modern:
      minVersion: "VersionTLS13"
    intermediate:
      minVersion: "VersionTLS12"
      cipherSuites:
        - "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256"
        - "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
        - "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384"
        - "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
        - "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305"
        - "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305"
http:
  middlewares:
    authelia:
      forwardAuth:
        address: https://authelia:9091/api/verify?rd=https://auth.example.com
        trustForwardHeader: true
        authResponseHeaders:
          - "Remote-User"
          - "Remote-Groups"
          - "Remote-Email"
          - "Remote-Name"
        tls:
          ca: /certificates/ca.public.crt
          cert: /certificates/traefik.public.crt
          key: /certificates/traefik.private.pem
    authelia-basic:
      forwardAuth:
        address: https://authelia:9091/api/verify?auth=basic
        trustForwardHeader: true
        authResponseHeaders:
          - "Remote-User"
          - "Remote-Groups"
          - "Remote-Email"
          - "Remote-Name"
        tls:
          ca: /certificates/ca.public.crt
          cert: /certificates/traefik.public.crt
          key: /certificates/traefik.private.pem
  routers:
    traefik:
      rule: Host(`traefik.example.com`)
      entryPoints: websecure
      service: api@internal
      middlewares:
        - authelia@file
      tls:
        options: modern@file
        certResolver: default
        domains:
          - main: "example.com"
            sans:
              - "*.example.com"
    whoami:
      rule: Host(`whoami.example.com`)
      entryPoints: websecure
      service: whoami-net@docker
      middlewares:
        - authelia@file
      tls:
        options: modern@file
        certResolver: default
        domains:
          - main: "example.com"
            sans:
              - "*.example.com"
    nextcloud:
      rule: Host(`nextcloud.example.com`)
      entryPoints: websecure
      service: nextcloud-net@docker
      middlewares:
        - authelia@file
      tls:
        options: modern@file
        certResolver: default
        domains:
          - main: "example.com"
            sans:
              - "*.example.com"
    heimdall:
      rule: Host(`heimdall.example.com`)
      entryPoints: websecure
      service: heimdall-net@docker
      middlewares:
        - authelia-basic@file
      tls:
        options: modern@file
        certResolver: default
        domains:
          - main: "example.com"
            sans:
              - "*.example.com"
    authelia:
      rule: Host(`auth.example.com`)
      entryPoints: websecure
      service: authelia@file
      tls:
        options: modern@file
        certResolver: default
        domains:
          - main: "example.com"
            sans:
              - "*.example.com"
  services:
    authelia:
      loadBalancer:
        servers:
          - url: https://authelia:9091/
        serversTransport: autheliaMutualTLS
  serversTransports:
    autheliaMutualTLS:
      certificates:
        - certFile: /certificates/traefik.public.crt
          keyFile: /certificates/traefik.private.pem
      rootCAs:
        - /certificates/ca.public.crt
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
