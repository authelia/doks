---
title: "HashiCorp Vault"
description: "Integrating HashiCorp Vault with Authelia via OpenID Connect."
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

{{< alert icon="⚠️" context="danger" >}}
This documentation is maintained by the community, it is not guaranteed to be complete or up-to-date. If you find an
error with this documentation please either make a pull request or start a GitHub Discussion.
{{< /alert >}}

## Tested Versions

- Authelia: v4.35.5
- HashiCorp Vault: 1.8.1

## Before You Begin

You are required to utilize a unique client id and a unique and random client secret for all [OpenID Connect] relying
parties. You should not use the client secret in this example, you should randomly generate one yourself. You may also
choose to utilize a different client id, it's completely up to you.

This example makes the following assumptions:

- **Application Root URL:** `https://vault.example.com`
- **Authelia Root URL:** `https://auth.example.com`
- **Client ID:** `vault`
- **Client Secret:** `vault_client_secret`

## Configuration

### Application

To configure [HashiCorp Vault] to utilize Authelia as an [OpenID Connect] Provider please see the links in the
[see also](#see-also) section.

### Authelia

The following YAML configuration is an example **Authelia**
[client configuration](../../../configuration/identity-providers/open-id-connect.md#clients) for use with [HashiCorp Vault]
which will operate with the above example:

```yaml
- id: vault
  secret: vault_client_secret
  public: false
  authorization_policy: two_factor
  scopes:
    - openid
    - profile
    - groups
    - email
  redirect_uris:
    - https://vault.example.com/oidc/callback
    - https://vault.example.com/ui/vault/auth/oidc/oidc/callback
  userinfo_signing_algorithm: none
```

## See Also

- [HashiCorp Vault JWT/OIDC Auth Documentation](https://www.vaultproject.io/docs/auth/jwt)
- [HashiCorp Vault OpenID Connect Providers Documentation](https://www.vaultproject.io/docs/auth/jwt/oidc_providers)

[HashiCorp Vault]: https://www.vaultproject.io/
[OpenID Connect]: ../../openid-connect/introduction.md
