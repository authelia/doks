---
title: "Prologue"
description: "An introduction into configuring Authelia."
lead: "An introduction into configuring Authelia."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "prologue"
weight: 100100
toc: true
---

## Validation

Authelia validates the configuration when it starts. This process checks multiple factors including configuration keys
that don't exist, configuration keys that have changed, the values of the keys are valid, and that a configuration
key isn't supplied at the same time as a secret for the same configuration option.

You may also optionally validate your configuration against this validation process manually by using the
`authelia validate-config` command. This command is useful prior to upgrading to prevent configuration changes from
impacting downtime in an upgrade. This process does not validate integrations, it only checks that your configuration
syntax is valid.

```bash
$ authelia validate-config --config configuration.yml
```
