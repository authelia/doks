---
title: "Password Policy"
description: "Password Policy Configuration"
lead: "An introduction into configuring Authelia."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "security"
weight: 104400
toc: true
---

_Authelia_ allows administrators to configure an enforced password policy.

## Configuration

```yaml
password_policy:
  standard:
    enabled: false
    min_length: 8
    max_length: 0
    require_uppercase: false
    require_lowercase: false
    require_number: false
    require_special: false
  zxcvbn:
    enabled: false
```

## Options

### standard

This section allows you to enable standard security policies.

#### enabled

{{< confkey type="boolean" default="false" required="no" >}}

Enables standard password policy.

#### min_length

{{< confkey type="integer" default="8" required="no" >}}

Determines the minimum allowed password length.

#### max_length

{{< confkey type="integer" default="0" required="no" >}}

Determines the maximum allowed password length.

#### require_uppercase

{{< confkey type="boolean" default="false" required="no" >}}

Indicates that at least one UPPERCASE letter must be provided as part of the password.

#### require_lowercase

{{< confkey type="boolean" default="false" required="no" >}}

Indicates that at least one lowercase letter must be provided as part of the password.

#### require_number

{{< confkey type="boolean" default="false" required="no" >}}

Indicates that at least one number must be provided as part of the password.

#### require_special

{{< confkey type="boolean" default="false" required="no" >}}

Indicates that at least one special character must be provided as part of the password.

### zxcvbn

This password policy enables advanced password strength metering, using [zxcvbn](https://github.com/dropbox/zxcvbn).

Note that this password policy do not restrict the user's entry it just gives the user feedback as to how strong their
password is.

#### enabled

{{< confkey type="boolean" default="false" required="no" >}}

_**Important Note:** only one password policy can be applied at a time._

Enables zxcvbn password policy.
