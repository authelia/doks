---
title: "OpenID Connect"
description: "Authelia OpenID Connect Implementation"
lead: "The OpenID Connect Provider role is a very useful but complex feature to enhance interoperability of Authelia with other products. "
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
roadmap:
  parent: "active"
weight: 220
toc: true
---

We have decided to implement [OpenID Connect] as a beta feature, it's suggested you only utilize it for testing and
providing feedback, and should take caution in relying on it in production as of now. [OpenID Connect] and it's related
endpoints are not enabled by default unless you specifically configure the [OpenID Connect] section.

As [OpenID Connect] is fairly complex (the [OpenID Connect] Provider role especially so) it's intentional that it is
both a beta and that the implemented features are part of a thoughtful roadmap. Items that are not immediately obvious
as required (i.e. bug fixes or spec features), will likely be discussed in team meetings or on GitHub issues before being
added to the list. We want to implement this feature in a very thoughtful way in order to avoid security issues.

## Stages

This section represents the stages involved in implementation of this feature. The stages are either in order of
implementation due to there being an underlying requirement to implement them in this order, or in a rough order
due to how important or difficult to implement they are.

### Beta 1

{{< roadmap-status stage="complete" version="v4.29.0" >}}

Feature List:

- [User Consent](https://openid.net/specs/openid-connect-core-1_0.html#Consent)
- [Authorization Code Flow](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps)
- [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html)
- [RS256 Signature Strategy](https://datatracker.ietf.org/doc/html/rfc7518#section-3.1)
- Per Client Scope/Grant Type/Response Type Restriction
- Per Client Authorization Policy (1FA/2FA)
- Per Client List of Valid Redirection URI's
- [Confidential Client Type](https://datatracker.ietf.org/doc/html/rfc6749#section-2.1)

### Beta 2

{{< roadmap-status stage="complete" version="v4.30.0" >}}

Feature List:

- [Userinfo Endpoint](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo)
- Parameter Entropy
- Token/Code Lifespan
- Client Debug Messages
- Client Audience
- [Public Client Type](https://datatracker.ietf.org/doc/html/rfc6749#section-2.1)

### Beta 3

{{< roadmap-status stage="complete" version="v4.34.0" >}}

Feature List:

- [Proof Key Code Exchange (PKCE)](https://datatracker.ietf.org/doc/html/rfc7636) for Authorization Code Flow
- Claims:
  - `preferred_username` - sending the username in this claim instead of the `sub` claim.

### Beta 4

{{< roadmap-status stage="complete" version="v4.35.0" >}}

Feature List:

- Persistent Storage
  - Tokens
  - Auditable Information
  - Subject to User Mapping
- Opaque UUID v4's for subject identifiers
- Support for Pairwise and Plain Subject Identifier types as per [OpenID Connect Core (Subject Identifier Types)](https://openid.net/specs/openid-connect-core-1_0.html#SubjectIDTypes)
- Claims:
  - `sub` - replace username with opaque random UUID v4
  - `amr` - authentication method references as per [RFC8176](https://datatracker.ietf.org/doc/html/rfc8176)
  - `azp` - authorized party as per [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html#IDToken)
  - `client_id` - the Client ID as per [RFC8693](https://datatracker.ietf.org/doc/html/rfc8693/#section-4.3)
- Cross Origin Resource Sharing (CORS):
  - Automatically allow all cross-origin requests to the discovery endpoints
  - Automatically allow all cross-origin requests to the JSON Web Keys endpoint
  - Optionally allow cross-origin requests to the other endpoints individually

### Beta 5

{{< roadmap-status >}}

Feature List:

- Prompt Handling
- Display Handling

### Beta 6

{{< roadmap-status >}}

Feature List:

- [Back-Channel Logout](https://openid.net/specs/openid-connect-backchannel-1_0.html)
- Revoke Tokens on User Logout or Expiration
- [JSON Web Key Rotation](https://openid.net/specs/openid-connect-messages-1_0-20.html#rotate.sig.keys)
- Hashed Client Secrets

### General Availability

{{< roadmap-status >}}

Feature List:

- Enable by Default
- Only after all previous stages are checked for bugs

### Miscellaneous

This stage lists features which individually do not fit into a specific stage and may or may not be implemented.

#### Front-Channel Logout

{{< roadmap-status >}}

See the [OpenID Connect] website for the [Front-Channel Logout specification](https://openid.net/specs/openid-connect-frontchannel-1_0.html).

#### OAuth 2.0 Authorization Server Metadata

{{< roadmap-status stage="complete" version="v4.34.0" >}}

See the [IETF Specification RFC8414](https://datatracker.ietf.org/doc/html/rfc8414) for more information.

#### OpenID Connect Session Management

{{< roadmap-status >}}

See the [OpenID Connect] website for the [OpenID Connect Session Management specification](https://openid.net/specs/openid-connect-session-1_0-17.html).

#### End-User Scope Grants

{{< roadmap-status >}}

Allow users to choose which scopes they grant.

#### Client RBAC

{{< roadmap-status >}}

Allow clients to be configured with a list of users and groups who have access to them.

#### Preferred Username Claim

{{< roadmap-status stage="complete" version="v4.33.2" >}}

The `preferred_username` claim was missing and was fixed.

[OpenID Connect]: https://openid.net/connect/
