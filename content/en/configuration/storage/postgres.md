---
title: "PostgreSQL"
description: "PostgreSQL Configuration"
lead: "The PostgreSQL storage provider."
date: 2022-03-20T12:52:27+11:00
lastmod: 2022-06-03T10:43:55+10:00
draft: false
images: []
menu:
  configuration:
    parent: "storage"
weight: 107400
toc: true
aliases:
  - /docs/configuration/storage/postgres.html
---

## Version support

See [PostgreSQL support](https://www.postgresql.org/support/versioning/) for the versions supported by PostgreSQL. We
recommend the *current minor* version of one of the versions supported by PostgreSQL.

The versions of PostgreSQL that should be supported by Authelia are:

* 14
* 13
* 12
* 11
* 10
* 9.6

## Configuration

```yaml
storage:
  encryption_key: a_very_important_secret
  postgres:
    host: 127.0.0.1
    port: 5432
    database: authelia
    schema: public
    username: authelia
    password: mypassword
    ssl:
      mode: disable
      root_certificate: /path/to/root_cert.pem
      certificate: /path/to/cert.pem
      key: /path/to/key.pem
```

## Options

### encryption_key

See the [encryption_key docs](introduction.md#encryption_key).

### host

{{< confkey type="string" required="yes" >}}

The database server host.

If utilising an IPv6 literal address it must be enclosed by square brackets and quoted:

```yaml
host: "[fd00:1111:2222:3333::1]"
```

### port

{{< confkey type="integer" default="5432" required="no" >}}

The port the database server is listening on.

### database

{{< confkey type="string" required="yes" >}}

The database name on the database server that the assigned [user](#username) has access to for the purpose of
__Authelia__.

### schema

{{< confkey type="string" default="public" required="no" >}}

The database schema name to use on the database server that the assigned [user](#username) has access to for the purpose
of __Authelia__. By default this is the public schema.

### username

{{< confkey type="string" required="yes" >}}

The username paired with the password used to connect to the database.

### password

{{< confkey type="string" required="yes" >}}

The password paired with the username used to connect to the database. Can also be defined using a
[secret](../methods/secrets.md) which is also the recommended way when running as a container.

We recommend generating a random string with 64 characters or more for this purposes which can be done by following the
[Generating a Random Alphanumeric String](../miscellaneous/guides.md#generating-a-random-alphanumeric-string)
guide.

### timeout

{{< confkey type="duration" default="5s" required="no" >}}

The SQL connection timeout.

### ssl

#### mode

{{< confkey type="string" default="disable" required="no" >}}

SSL mode configures how to handle SSL connections with Postgres.
Valid options are 'disable', 'require', 'verify-ca', or 'verify-full'.
See the [PostgreSQL Documentation](https://www.postgresql.org/docs/12/libpq-ssl.html)
or [pgx - PostgreSQL Driver and Toolkit Documentation](https://pkg.go.dev/github.com/jackc/pgx?tab=doc)
for more information.

#### root_certificate

{{< confkey type="string" required="no" >}}

The optional location of the root certificate file encoded in the PEM format for validation purposes.

#### certificate

{{< confkey type="string" required="no" >}}

The optional location of the certificate file encoded in the PEM format for validation purposes.

#### key

{{< confkey type="string" required="no" >}}

The optional location of the key file encoded in the PEM format for authentication purposes.
