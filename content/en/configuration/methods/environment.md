---
title: "Environment"
description: "Environment Configuration Method"
lead: "Authelia has a layered configuration model. This section describes how to implement the environment configuration."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "methods"
weight: 101300
toc: true
---

Environment variables are applied after the configuration file meaning anything specified as part of the environment
overrides the configuration files.

_**Please Note:** It is not possible to configure the access control rules section or OpenID Connect identity provider
clients section using environment variables at this time._

## Prefix

The environment variables must be prefixed with `AUTHELIA_`. All environment variables that start with this prefix must
be for configuration. Any supplied environment variables that have this prefix and are not meant for configuration will
likely result in an error or even worse misconfiguration.

### Kubernetes

_**Important Note:**_ There are compatability issues with Kubernetes and this particular configuration option. You must
ensure you have the `enableServiceLinks: false` setting in your pod spec.

Here an excerpt from a manifest as an example:

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: authelia
  namespace: authelia
  labels:
    app.kubernetes.io/instance: authelia
    app.kubernetes.io/name: authelia
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/instance: authelia
      app.kubernetes.io/name: authelia
  template:
    metadata:
      labels:
        app.kubernetes.io/instance: authelia
        app.kubernetes.io/name: authelia
    spec:
      enableServiceLinks: false
      containers:
        - name: authelia
          image: authelia/authelia:latest
          command:
            - authelia
          args:
            - '--config=/configuration.yaml'
            - '--config=/configuration.acl.yaml'
```

See the [PodSpec API documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#podspec-v1-core)
for more information.

## Mapping

Configuration options are mapped by their name. Levels of indentation / subkeys are replaced by underscores.

For example this YAML configuration:

```yaml
log:
  level: info
server:
  read_buffer_size: 4096
```

Can be replaced by this environment variable configuration:

```bash
AUTHELIA_LOG_LEVEL=info
AUTHELIA_SERVER_READ_BUFFER_SIZE=4096
```


