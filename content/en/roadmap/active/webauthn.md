---
title: "Webauthn"
description: "Authelia Webauthn Implementation"
lead: "An introduction into the Authelia roadmap."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  roadmap:
    parent: "active"
weight: 210
toc: true
---

Webauthn requires urgent implementation as it is being deprecated by Chrome. It is a modern evolution of the U2F/FIDO
protocol and is very similar in many ways. It even includes a backwards compatability layer which allows a previously
registered U2F device to be used with the protocol to authenticate.

## Stages

This section represents the stages involved in implementation of this feature. The stages are either in order of
implementation due to there being an underlying requirement to implement them in this order, or in their likely order
due to how important or difficult to implement they are.

### Initial Implementation

{{< roadmap-status stage="complete" version="v4.34.0" >}}

Implement Webauthn as a replacement for U2F with U2F backwards compatibility.

|                       Setting                       |     Value      |                                                                Effect                                                                |
|:---------------------------------------------------:|:--------------:|:------------------------------------------------------------------------------------------------------------------------------------:|
|               Conveyancing Preference               |    indirect    | Configurable: ask users to permit collection of the AAGUID, this is like a model number, this GUID will be stored in the SQL storage |
|                  User Verification                  |   preferred    |                           Configurable: ask the browser to prompt for the users PIN or other verification                            |
| Discoverable Credentials (Resident Key Requirement) |  discouraged   |                                       See the [passwordless login stage](#passwordless-login)                                        |
|              Authenticator Attachment               | cross-platform |                                   See the [platform authenticator stage](#platform-authenticator)                                    |
|              Multi-Device Registration              |  unavailable   |                                see the [multi device registration stage](#multi-device-registration)                                 |

### Multi Device Registration

{{< roadmap-status stage="waiting" >}}

Implement multi device registration as part of the user interface. This is technically implemented for the most part in
the backend, it's just the public facing interface elements remaining.

### Platform Authenticator

{{< roadmap-status >}}

Implement Webauthn Platform Authenticators so that people can use things like Windows Hello, TouchID, or an Android
Security Key.

### Passwordless Login

{{< roadmap-status >}}

Implement Webauthn Passwordless Login. This


