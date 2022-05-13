---
title: "Duo / Mobile Push"
description: "Configuring Duo"
lead: ""
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "second-factor"
weight: 103200
toc: true
---

Authelia supports mobile push notifications relying on [Duo].

Follow the instructions in the dedicated [documentation](../features/2fa/push-notifications.md) for instructions on how
to set up push notifications in Authelia.

**Note:** The configuration options in the following sections are noted as required. They are however only required when
you have this section defined. i.e. if you don't wish to use the [Duo] push notifications you can just not define this
section of the configuration.

## Configuration

The configuration is as follows:
```yaml
duo_api:
  disable: false
  hostname: api-123456789.example.com
  integration_key: ABCDEF
  secret_key: 1234567890abcdefghifjkl
  enable_self_enrollment: false
```

The secret key is shown as an example, you also have the option to set it using an environment variable as described
[here](./secrets.md).

## Options

### Disable

{{< confkey type="boolean" default="false" required="no" >}}

Disables Duo. If the hostname, integration_key, and secret_key are all empty strings or undefined this is automatically
true.

### hostname

{{< confkey type="string" required="yes" >}}

The [Duo] API hostname supplied by [Duo].

### integration_key

{{< confkey type="string" required="yes" >}}

The non-secret [Duo] integration key. Similar to a client identifier.

### secret_key

{{< confkey type="string" required="yes" >}}

The secret [Duo] key used to verify your application is valid.

### enable_self_enrollment

{{< confkey type="boolean" default="false" required="no" >}}

Enables [Duo] device self-enrollment from within the Authelia portal.

[Duo]: https://duo.com/
