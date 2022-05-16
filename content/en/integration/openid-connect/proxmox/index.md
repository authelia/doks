---
title: "Proxmox"
description: "Integrating Proxmox with Authelia via OpenID Connect."
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

- Authelia: v4.33.2
- Proxmox: 7.1-10

## Before You Begin

You are required to utilize a unique client id and a unique and random client secret for all [OpenID Connect] relying
parties. You should not use the client secret in this example, you should randomly generate one yourself. You may also
choose to utilize a different client id, it's completely up to you.

This example makes the following assumptions:

- **Application Root URL:** `https://proxmox.example.com`
- **Authelia Root URL:** `https://auth.example.com`
- **Client ID:** `proxmox`
- **Client Secret:** `proxmox_client_secret`

## Configuration

### Application

To configure [Proxmox] to utilize Authelia as an [OpenID Connect] Provider:

1. Visit Datacenter
2. Visit Permission
3. Visit Realms
4. Add an OpenID Connect Server
5. Configure the following:
   1. Issuer URL: `https://auth.example.com`
   2. Realm: anything you wish
   3. Client ID: `proxmox`
   4. Client Key: `proxmox_client_secret`
   5. Username Claim `preferred_username`
   6. Scopes: `openid profile email`
   7. Enable *Autocreate Users* if you want users to automatically be created in [Proxmox].

{{< figure src="proxmox.gif" alt="Proxmox" width="736" style="padding-right: 10px" >}}

### Authelia

The following YAML configuration is an example **Authelia**
[client configuration](../../../configuration/identity-providers/open-id-connect.md#clients) for use with [Proxmox]
which will operate with the above example:

```yaml
- id: proxmox
  secret: proxmox_client_secret
  public: false
  authorization_policy: two_factor
  scopes:
    - openid
    - profile
    - email
  redirect_uris:
    - https://proxmox.example.com
  userinfo_signing_algorithm: none
```

## See Also

- [Proxmox User Management Documentation](https://pve.proxmox.com/wiki/User_Management)

[Proxmox]: https://www.proxmox.com/
[OpenID Connect]: ../../openid-connect/introduction.md
