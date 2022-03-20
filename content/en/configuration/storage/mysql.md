---
title: "MySQL"
description: "MySQL Configuration"
lead: "The MySQL storage provider."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  configuration:
    parent: "storage"
weight: 107400
toc: true
---

## Configuration

```yaml
storage:
  encryption_key: a_very_important_secret
  mysql:
    host: 127.0.0.1
    port: 3306
    database: authelia
    username: authelia
    password: mypassword
    timeout: 5s
```

## Options

### encryption_key
See the [encryption_key docs](./index.md#encryption_key).

### host

{{< confkey type="string" default="localhost" required="no" >}}

The database server host.

If utilising an IPv6 literal address it must be enclosed by square brackets and quoted:
```yaml
host: "[fd00:1111:2222:3333::1]"
```

### port

{{< confkey type="integer" default="3306" required="no" >}}

The port the database server is listening on.

### database

{{< confkey type="string" required="yes" >}}

The database name on the database server that the assigned [user](#username) has access to for the purpose of
**Authelia**.

### username

{{< confkey type="string" required="yes" >}}

The username paired with the password used to connect to the database.

### password

{{< confkey type="string" required="yes" >}}

The password paired with the username used to connect to the database. Can also be defined using a
[secret](../methods/secrets.md) which is also the recommended way when running as a container.

### timeout

{{< confkey type="duration" default="5s" required="no" >}}

The SQL connection timeout.
