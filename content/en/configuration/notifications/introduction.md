---
title: "Notifications"
description: "Notifications Configuration"
lead: "Authelia sends messages to users in order to verify their identity. This section describes how to configure this."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "notifications"
weight: 108100
toc: true
---

Authelia sends messages to users in order to verify their identity.

## Configuration

```yaml
notifier:
  disable_startup_check: false
  filesystem: {}
  smtp: {}
```

## Options

### disable_startup_check

{{< confkey type="boolean" default="false" required="no" >}}

The notifier has a startup check which validates the specified provider
configuration is correct and will be able to send emails. This can be
disabled with the `disable_startup_check` option:

### filesystem

The [filesystem](file.md) provider.

### smtp

The [smtp](smtp.md) provider.
