---
title: "Reference: Authelia Scripts"
description: "This section covers the authelia-scripts tool."
lead: "This section covers the authelia-scripts tool."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  contributing:
    parent: "development"
weight: 290
toc: true
---

Authelia comes with a set of dedicated scripts to perform a broad range of operations such as building the distributed
version of Authelia, building the Docker image, running suites, testing the code, etc...

Those scripts become available after sourcing the bootstrap.sh script with:

```console
$ source bootstrap.sh
```

Then, you can access the scripts usage by running the following command:

```console
$ authelia-scripts --help
```

For instance, you can build Authelia (Go binary and frontend) with:

```console
$ authelia-scripts build
```

Or build the official Docker image with:

```console
$ authelia-scripts docker build
```

Or start the *Standalone* suite with:

```console
$ authelia-scripts suites setup Standalone
```

You will find more information in the scripts usage helpers.
