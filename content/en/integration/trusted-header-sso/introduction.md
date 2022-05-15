---
title: "Trusted Header SSO"
description: "Trusted Header SSO Integration"
lead: "An introduction into integrating Authelia with an application which implements authentication via trusted headers."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "trusted-header-sso"
weight: 610
toc: true
---

Authelia will respond to requests via the forward authentication flow with specific headers that can be utilized by some
applications to perform authentication. This section of the documentation discusses how to integrate these products with
this model.

Please see the [proxy integration](../proxies/introduction.md) for more information on how to return these headers to
the application.

## Terminology

This authentication method is referred to by many names; notably `trusted header authentication`,
`header authentication`, `header sso`, and probably many more.

## Specifics

The headers are not intended to be returned to a browser, instead these headers are meant for internal communications
only. These headers are returned to the reverse proxy and then injected into the request which the reverse proxy makes
to the application's http endpoints.

This allows these applications to decide if they wish to trust these headers, and if they do trust them, perform some
form of authentication flow. This flow usually takes the form of automatically logging users into the application,
however it can vary depending on how the application decides to do this.

## Response Headers

The following table represents the response headers that Authelia's `/api/verify` endpoint returns which can be
forwarded over a trusted network via the reverse proxy when using the forward authentication flow.

|    Header     |      Description / Notes       |      Example       |
|:-------------:|:------------------------------:|:------------------:|
|  Remote-User  |       The users username       |        john        |
| Remote-Groups | The groups the user belongs to |     admin,dev      |
|  Remote-Name  |     The users display name     |     John Smith     |
| Remote-Email  |    The users email address     | jsmith@example.com |
