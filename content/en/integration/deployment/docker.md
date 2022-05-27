---
title: "Docker"
description: "A guide on installing Authelia in Docker."
lead: "This is one of the primary ways we deliver Authelia to users and the recommended path."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "deployment"
weight: 220
toc: true
---

The [Docker] container is deployed with the following image names:

- [authelia/authelia](https://hub.docker.com/r/authelia/authelia)
- [docker.io/authelia/authelia](https://hub.docker.com/r/authelia/authelia)
- [ghcr.io/authelia/authelia](https://github.com/authelia/authelia/pkgs/container/authelia)

## Docker Compose

We provide two main [Docker Compose] bundles which can be utilized to help test _Authelia_ or can be adapted into your
existing [Docker Compose].

- [lite bundle](https://github.com/authelia/authelia/tree/master/examples/compose/lite)
- [local bundle](https://github.com/authelia/authelia/tree/master/examples/compose/local)

### Example

The following is an example [Docker Compose] deployment with just _Authelia_ and no bundled applications or proxies.

It expects the following:

- The file `./data/authelia/config/configuration.yml` is present and the configuration file.
- The files `./data/authelia/secrets/*` exist and contain the relevant secrets.
- You're using PostgreSQL.
- You have an external network named `net` which is in bridge mode.

```yaml
version: "3.8"
secrets:
  jwt:
    file: ${PWD}/data/authelia/secrets/jwt
  storage:
    file: ${PWD}/data/authelia/secrets/storage
  storageEnc:
    file: ${PWD}/data/authelia/secrets/storageEnc
  session:
    file: ${PWD}/data/authelia/secrets/session
  oidcHMACSecret:
    file: ${PWD}/data/authelia/secrets/oidcHMACSecret
  oidcPrivateKey:
    file: ${PWD}/data/authelia/secrets/oidcPrivateKey
services:
  authelia:
    container_name: authelia
    image: docker.io/authelia/authelia:latest
    restart: unless-stopped
    networks:
      net:
        aliases: []
    expose:
      - 9091
    secrets: [jwt, session, storage, storageEnc, oidcHMACSecret, oidcPrivateKey]
    environment:
      AUTHELIA_JWT_SECRET_FILE: /run/secrets/jwt
      AUTHELIA_SESSION_SECRET_FILE: /run/secrets/session
      AUTHELIA_STORAGE_POSTGRES_PASSWORD_FILE: /run/secrets/storage
      AUTHELIA_STORAGE_ENCRYPTION_KEY_FILE: /run/secrets/storageEnc
      AUTHELIA_IDENTITY_PROVIDERS_OIDC_ISSUER_PRIVATE_KEY_FILE: /run/secrets/oidcPrivateKey
      AUTHELIA_IDENTITY_PROVIDERS_OIDC_HMAC_SECRET_FILE: /run/secrets/oidcHMACSecret
    volumes:
      - ${PWD}/data/authelia/config:/config
networks:
  net:
    external: true
    name: net
```
