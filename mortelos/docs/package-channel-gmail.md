---
title: "Gmail Channel"
nav_title: "Gmail channel"
slug: "package-channel-gmail"
version: "0"
description: "Gmail channel driver for inbound and outbound workspace mail."
section: "packages"
order: 58
status: "mvp"
audience: "developers"
package: "mortelos/channel-gmail"
canonical_path: "/docs/0/package-channel-gmail"
last_verified: "2026-06-04"
public: true
---

# Gmail Channel

`mortelos/channel-gmail` is the Gmail channel driver for MortelOS.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Inbound mail | Poll Gmail and process inbound mail into MortelOS channel flows. |
| Outbound mail | Execute send-mail actions through Gmail. |
| Classification | Classify inbound mail before local workflow handling. |
| OAuth callback | Handle Google OAuth callback wiring for the channel. |

## Install

```bash
composer require mortelos/channel-gmail
```

## Boundaries

The package owns the Gmail driver, API client, polling jobs and send-mail action. The host app owns Google credentials, tenant policy, message retention choices and local workflow decisions.

