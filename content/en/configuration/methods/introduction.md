---
title: "Methods"
description: "Methods of Configuration"
lead: "Authelia has a layered configuration model. This section describes how to implement configuration."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "methods"
weight: 101100
toc: true
---

## Layers

Authelia has several methods of configuration available to it. The order of precedence is as follows:

1. [Secrets](secrets.md)
2. [Environment Variables](environment.md)
3. [Files](files.md) (in order of them being specified)

This order of precedence puts higher weight on things higher in the list. This means anything specified in the
[files](files.md) is overridden by [environment variables](environment.md) if specified, and anything specified by
[environment variables](environment.md) is overridden by [secrets](secrets.md) if specified.

## Documentation

We document the configuration in two ways:

1. The configuration yaml default has comments documenting it. All documentation lines start with `##`. Lines starting
   with a single `#` are yaml configuration options which are commented to disable them or as examples.

2. This documentation site. Generally each section of the configuration is in its own section of the documentation
   site. Each configuration option is listed in its relevant section as a heading, under that heading generally are two
   or three colored labels.
  - The `type` label is purple and indicates the yaml value type of the variable. It optionally includes some
    additional information in parentheses.
  - The `default` label is blue and indicates the default value if you don't define the option at all. This is not the
    same value as you will see in the examples in all instances, it is the value set when blank or undefined.
  - The `required` label changes color. When required it will be red, when not required it will be green, when the
    required state depends on another configuration value it is yellow.

## Validation

Authelia validates the configuration when it starts. This process checks multiple factors including configuration keys
that don't exist, configuration keys that have changed, the values of the keys are valid, and that a configuration
key isn't supplied at the same time as a secret for the same configuration option.

You may also optionally validate your configuration against this validation process manually by using the validate-config
option with the Authelia binary as shown below. Keep in mind if you're using [secrets](./secrets.md) you will have to
manually provide these if you don't want to get certain validation errors (specifically requesting you provide one of
the secret values). You can choose to ignore them if you know what you're doing. This command is useful prior to
upgrading to prevent configuration changes from impacting downtime in an upgrade. This process does not validate
integrations, it only checks that your configuration syntax is valid.

```bash
$ authelia validate-config --config configuration.yml
```
