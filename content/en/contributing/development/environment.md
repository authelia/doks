---
title: "Environment"
description: "How to configure your development environment."
lead: "This section covers the environment we recommend for development."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  contributing:
    parent: "development"
weight: 220
toc: true
---

**Authelia** and its development workflow can be tested with Docker and docker-compose on Linux.

In order to deploy the current version of Authelia locally, run the following command and follow the instructions of
bootstrap.sh:

```console
$ source bootstrap.sh
```

Then, start the *Standalone* [suite].
```console
$ authelia-scripts suites setup Standalone
```

A [suite] is kind of a virtual environment for running Authelia in a complete ecosystem. If you want more details please
read the related [documentation](./integration-suites.md).

## FAQ

### What version of Docker and docker-compose should I use?

Here are the versions used for testing in Buildkite:

```console
$ docker --version
Docker version 20.10.8, build 3967b7d

$ docker-compose --version
docker-compose version 1.28.0, build unknown
```

### How can I serve my application under example.com?

Don't worry, you don't need to own the domain *example.com* to test Authelia. Copy the following lines in
your `/etc/hosts`.

```
192.168.240.100 home.example.com
192.168.240.100 login.example.com
192.168.240.100 singlefactor.example.com
192.168.240.100 public.example.com
192.168.240.100 secure.example.com
192.168.240.100 mail.example.com
192.168.240.100 mx1.mail.example.com
```

`192.168.240.100` is the IP attributed by Docker to the reverse proxy. Once added you can access the listed sub-domains
from your browser, and they will target the reverse proxy.

[suite]: ./integration-suites.md
