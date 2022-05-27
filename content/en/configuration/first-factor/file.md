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

Configuring _Authelia_ to use a file is done by specifying the path to the file in the configuration file.

```yaml
authentication_backend:
  disable_reset_password: false
  file:
    path: /config/users.yml
    password:
      algorithm: argon2id
      iterations: 1
      key_length: 32
      salt_length: 16
      parallelism: 8
      memory: 64
```

## Format

The format of the users file is as follows.

```yaml
users:
  john:
    displayname: "John Doe"
    password: "$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM"
    email: john.doe@authelia.com
    groups:
      - admins
      - dev
  harry:
    displayname: "Harry Potter"
    password: "$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM"
    email: harry.potter@authelia.com
    groups: []
  bob:
    displayname: "Bob Dylan"
    password: "$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM"
    email: bob.dylan@authelia.com
    groups:
      - dev
  james:
    displayname: "James Dean"
    password: "$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM"
    email: james.dean@authelia.com
```

This file should be set with read/write permissions as it could be updated by users resetting their passwords.

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

| Algorithm | Minimum | Default |                       Recommended                       |
|:---------:|:-------:|:-------:|:-------------------------------------------------------:|
| argon2id  |    1    |    3    | [See Recommendations](#recommended-parameters-argon2id) |
|  sha512   |  1000   |  50000  |                          50000                          |

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

## Passwords

The file contains hashed passwords instead of plain text passwords for security reasons.

You can use Authelia binary or docker image to generate the hash of any password. The hash-password command has many
tunable options, you can view them with the `authelia hash-password --help` command. For example if you wanted to improve
the entropy you could generate a 16 byte salt and provide it with the `--salt` flag.

Example: `authelia hash-password --salt abcdefghijklhijl -- "password"`.

For argon2id the salt must always be valid for base64 decoding (characters a through z, A through Z, 0 through 9, and +/).

Passwords passed to `hash-password` should be single quoted if using special characters to prevent parameter substitution.
For instance to generate a hash with the docker image just run:

```bash
$ docker run authelia/authelia:latest authelia hash-password -- "password"
Password hash: $argon2id$v=19$m=65536$3oc26byQuSkQqksq$zM1QiTvVPrMfV6BVLs2t4gM+af5IN7euO0VB6+Q8ZFs
```

You may also use the `--config` flag to point to your existing configuration. When used, the values defined in the
config will be used instead.

Full CLI Help Documentation:

```
Hash a password to be used in file-based users database. Default algorithm is argon2id.

Usage:
  authelia hash-password [flags] -- <password>

Flags:
  -c, --config strings    Configuration files
  -h, --help              help for hash-password
  -i, --iterations int    set the number of hashing iterations (default 3)
  -k, --key-length int    [argon2id] set the key length param (default 32)
  -m, --memory int        [argon2id] set the amount of memory param (in MB) (default 64)
  -p, --parallelism int   [argon2id] set the parallelism param (default 4)
  -s, --salt string       set the salt string
  -l, --salt-length int   set the auto-generated salt length (default 16)
  -z, --sha512            use sha512 as the algorithm (changes iterations to 50000, change with -i)
```

### Password hash algorithm

The default hash algorithm is Argon2id version 19 with a salt. Argon2id is currently considered the best hashing
algorithm, and in 2015 won the
[Password Hashing Competition](https://en.wikipedia.org/wiki/Password_Hashing_Competition). It benefits from
customizable parameters allowing the cost of computing a hash to scale into the future which makes it harder to
brute-force.

For backwards compatibility and user choice support for the SHA512 algorithm is still available. While it's a reasonable
hashing function given high enough iterations, as hardware improves it has a higher chance of being brute-forced.

Hashes are identifiable as Argon2id or SHA512 by their prefix of either `$argon2id$` and `$6$` respectively, as
described in this [wiki page](https://en.wikipedia.org/wiki/Crypt_(C)).

**Important Note:** When using argon2id Authelia will appear to remain using the memory allocated
to creating the hash. This is due to how [Go](https://golang.org/) allocates memory to the heap when
generating an argon2id hash. Go periodically garbage collects the heap, however this doesn't remove
the memory allocation, it keeps it allocated even though it's technically unused. Under memory
pressure the unused allocated memory will be reclaimed by the operating system, you can test
this on linux with:

```bash
$ stress-ng --vm-bytes $(awk '/MemFree/{printf "%d\n", $2 * 0.9;}' < /proc/meminfo)k --vm-keep -m 1
```

If this is not desirable we recommend investigating the following options in order of most to least secure:

1. Use the [LDAP](ldap.md) authentication provider instead
2. Adjusting the [memory](#memory) parameter
3. Changing the [algorithm](#algorithm)

### Password hash algorithm tuning

All algorithm tuning for Argon2id is supported. The only configuration variables that affect SHA512 are iterations and
salt length. The configuration variables are unique to the file authentication provider, thus they all exist in a key
under the file authentication configuration key called `password`. We have set what are considered as sane and
recommended defaults to cater for a reasonable system, if you're unsure about which settings to tune, please see the
parameters below, or for a more in depth understanding see the referenced documentation in
[Argon2 links](file.md#argon2-links).

#### Recommended Parameters: Argon2id

This table adapts the [RFC9106 Parameter Choice] recommendations to our configuration options:

|  Situation  | Iterations | Parallelism | Memory | Salt Size | Key Size |
|:-----------:|:----------:|:-----------:|:------:|:---------:|:--------:|
| Low Memory  |     3      |      4      |  64MB  |    16     |    32    |
| Recommended |     1      |      4      |  2GB   |    16     |    32    |

## Argon2 Links

- [RFC9106] (Argon2)
- [Go Documentation](https://godoc.org/golang.org/x/crypto/argon2)

[RFC9106]: https://www.rfc-editor.org/rfc/rfc9106.html
[RFC9106 Parameter Choice]: https://www.rfc-editor.org/rfc/rfc9106.html#section-4
