---
title: "Jira"
description: "Trusted Header SSO Integration for Jira"
lead: "This documentation is maintained by the community. This documentation is not guaranteed to be complete or up-to-date. If you find an error with this documentation please either make a pull request or start a GitHub Discussion."
date: 2022-03-19T04:53:05+00:00
lastmod: 2022-03-19T04:53:05+00:00
draft: false
images: []
menu:
  integration:
    parent: "trusted-header-sso"
weight: 621
toc: true
---

This is a guide on integration of **Authelia** and [Jira] via the trusted header SSO authentication.

As with all guides in this section it's important you read the [introduction](../introduction.md) first.

## Tested Versions

- Authelia: v4.35.5
- Jira: Unknown
- EasySSO: Unknown

## Before You Begin

This example makes the following assumptions:

- The user accounts with the same names already exist in [Jira].
- You have purchased the third-party plugin from the [Atlassian marketplace](https://marketplace.atlassian.com/apps/1212581/easy-sso-jira-kerberos-ntlm-saml?hosting=server&tab=overview)

## Configuration

To configure [Jira] to trust the `Remote-User` and `Remote-Email` header do the following:

1. Visit the Easy SSO plugin settings
2. Under HTTP configure the `Remote-User` header
3. Check the `Username` checkbox

[Jira]: https://www.atlassian.com/software/jira
