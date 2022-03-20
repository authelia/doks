---
title: "Kubernetes Documentation"
description: "Add better Kubernetes documentation."
lead: "While there is some documentation for Kubernetes, and several people have it working, better documentation is needed."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
roadmap:
  parent: "active"
weight: 250
toc: true
---

## Stages

This section represents the stages involved in implementation of this feature. The stages are either in order of
implementation due to there being an underlying requirement to implement them in this order, or in their likely order
due to how important or difficult to implement they are.

### Helm Chart

{{< roadmap-status stage="in-progress" >}}

Develop and release a Helm Chart which makes implementation on Kubernetes easy.

This is currently in progress and there is a [Helm Chart Repository](https://charts.authelia.com). This is considered
beta and the chart itself has a lot of work to go.

### Integration Documentation

{{< roadmap-status stage="in-progress" >}}

Provide some generalized integration documentation for Kubernetes.

### Kustomize

{{< roadmap-status >}}

Implement a Kustomize bundle people can utilize.
