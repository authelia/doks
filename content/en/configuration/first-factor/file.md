---
title: "File"
description: "File"
lead: "Authelia supports a file based first factor user provider. This section describes configuring this."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "first-factor"
weight: 102300
toc: true
---

## Configuration

```yaml
authentication_backend:
  disable_reset_password: false
  file:
    path: /config/users.yml
    password:
      algorithm: argon2id
      iterations: 3
      key_length: 32
      salt_length: 16
      parallelism: 4
      memory: 64
```

## Options

### path

{{< confkey type="string" required="yes" >}}

The path to the YAML file with the user details list.

### password

#### algorithm

{{< confkey type="string" default="argon2id" required="no" >}}

Controls the hashing algorithm used for hashing new passwords. Value must be one of `argon2id` or `sha512`.

#### iterations

{{< confkey type="integer" required="no" >}}

Controls the number of hashing iterations done by the other hashing settings.

| Algorithm | Minimum | Default |                                        Recommended                                         |
|:---------:|:-------:|:-------:|:------------------------------------------------------------------------------------------:|
| argon2id  |    1    |    3    | [See Recommendations](../../reference/guides/passwords.md#recommended-parameters-argon2id) |
|  sha512   |  1000   |  50000  |                                           50000                                            |

#### key_length

{{< confkey type="integer" default="32" required="no" >}}

This setting is specific to `argon2id` and unused with `sha512`. Sets the key length of the argon2 output. Minimum value
is `16`, with a recommended value of `32`.

#### salt_length

{{< confkey type="integer" default="16" required="no" >}}

Controls the length of the random salt added to each password before hashing. Minimum value is `8`, with a recommended
value of `16`. There is not a compelling reason to have this set to anything other than `16`.

#### parallelism

{{< confkey type="integer" default="4" required="no" >}}

This setting is specific to `argon2id` and unused with `sha512`. Sets the number of threads used when hashing passwords,
which affects the effective cost of hashing.

#### memory

{{< confkey type="integer" default="64" required="no" >}}

This setting is specific to `argon2id` and unused with `sha512`. Sets the amount of memory allocated to a single
password hashing action. This memory is released by go after the hashing process completes, however the operating system
may not reclaim it until it needs the memory which may make Authelia appear to be using more memory than it technically
is.

## Reference

A [reference guide](../../reference/guides/passwords.md) exists specifically for choosing password hashing values. This
section contains far more information than is practical to include in this configuration document. See the
[Passwords Reference Guide](../../reference/guides/passwords.md) for more information.

This guide contains examples such as the [Users YAML File](../../reference/guides/passwords.md#yaml-format).
