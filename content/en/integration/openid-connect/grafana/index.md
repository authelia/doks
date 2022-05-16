---
title: "Grafana"
description: "Integrating Grafana with Authelia via OpenID Connect."
lead: ""
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "openid-connect"
weight: 420
toc: true
---

{{< community-doc >}}

## Tested Versions

- Authelia: v4.35.5
- Grafana: 8.0.0

## Before You Begin

You are required to utilize a unique client id and a unique and random client secret for all [OpenID Connect] relying
parties. You should not use the client secret in this example, you should randomly generate one yourself. You may also
choose to utilize a different client id, it's completely up to you.

This example makes the following assumptions:

- **Application Root URL:** `https://grafana.example.com`
- **Authelia Root URL:** `https://auth.example.com`
- **Client ID:** `grafana`
- **Client Secret:** `grafana_client_secret`

## Configuration

### Application

To configure [Grafana] to utilize Authelia as an [OpenID Connect] Provider:

1. Add the following Generic OAuth configuration to the [Grafana] configuration:

```ruby
[auth.generic_oauth]
enabled = true
name = Authelia
icon = signin
client_id = grafana
client_secret = grafana_client_secret
scopes = openid profile email groups
empty_scopes = false
auth_url = https://auth.example.com/api/oidc/authorization
token_url = https://auth.example.com/api/oidc/token
api_url = https://auth.example.com/api/oidc/userinfo
login_attribute_path = preferred_username
groups_attribute_path = groups
name_attribute_path = name
use_pkce = true
```

### Authelia

The following YAML configuration is an example **Authelia**
[client configuration](../../../configuration/identity-providers/open-id-connect.md#clients) for use with [Grafana]
which will operate with the above example:

```yaml
- id: grafana
  secret: grafana_client_secret
  public: false
  authorization_policy: two_factor
  scopes:
    - openid
    - profile
    - groups
    - email
  redirect_uris:
    - https://grafana.example.com/login/generic_oauth
  userinfo_signing_algorithm: none
```

## See Also

- [Grafana OAuth Documentation](https://grafana.com/docs/grafana/latest/auth/generic-oauth/)

[Grafana]: https://grafana.com/
[OpenID Connect]: ../../openid-connect/introduction.md
