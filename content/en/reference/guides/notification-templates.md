---
title: "Notification Templates"
description: "A reference guide on overriding notification templates"
lead: "This section contains reference documentation for Authelia's notification templates."
date: 2022-03-20T12:52:27+11:00
lastmod: 2022-06-03T10:43:55+10:00
draft: false
images: []
menu:
  reference:
    parent: "guides"
weight: 220
toc: true
---

Authelia uses templates to generate the HTML and plaintext emails sent via the notification service. Each template has
two extensions; `.html` for HTML templates, and `.txt` for plaintext templates.

This guide is effectively documents the usage of the
[template_path](../../configuration/notifications/introduction.md#template_path) notification configuration option.

## Template Names

|       Template       |                                    Description                                    |
|:--------------------:|:---------------------------------------------------------------------------------:|
| IdentityVerification | Used to render notifications sent when registering devices or resetting passwords |
|    PasswordReset     |    Used to render notifications sent when password has successfully been reset    |

For example, to modify the `IdentityVerification` HTML template, if your
[template_path](../../configuration/notifications/introduction.md#template_path) was configured as
`/config/email_templates`, you would create the `/config/email_templates/IdentityVerification.html` file to override the
HTML `IdentityVerification` template.

## Placeholder Variables

In template files, you can use the following placeholders which are automatically injected into the templates:

|     Placeholder      |      Templates       |                                                                  Description                                                                   |
|:--------------------:|:--------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------:|
|   `{{ .LinkURL }}`   | IdentityVerification |                                            The URL associated with the notification if applicable.                                             |
|  `{{ .LinkText }}`   | IdentityVerification |                                 The display value for the URL associated with the notification if applicable.                                  |
|    `{{ .Title }}`    |         All          | A predefined title for the email. <br> It will be `"Reset your password"` or `"Password changed successfully"`, depending on the current step. |
| `{{ .DisplayName }}` |         All          |                                                     The name of the user, i.e. `John Doe`                                                      |
|  `{{ .RemoteIP }}`   |         All          |                                      The remote IP address (client) that initiated the request or event.                                       |

## Examples

This is a basic example:

```html
<body>
  <h1>{{ .Title }}</h1>
  Hi {{ .DisplayName }}<br/>
  This email has been sent to you in order to validate your identity.
  Click <a href="{{ .LinkURL }}">here</a> to change your password.
</body>
```

Some Additional examples for specific purposes can be found in the
[examples directory on GitHub](https://github.com/authelia/authelia/tree/master/examples/templates/notifications).

## Original Templates

The original template content can be found on
[GitHub](https://github.com/authelia/authelia/tree/master/internal/templates).
