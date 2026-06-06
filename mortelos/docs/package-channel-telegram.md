---
title: "Telegram Channel"
nav_title: "Telegram channel"
slug: "package-channel-telegram"
version: "0"
description: "Telegram channel driver for bidirectional Bot API communication."
section: "packages"
order: 62
status: "mvp"
audience: "developers"
package: "mortelos/channel-telegram"
canonical_path: "/docs/0/package-channel-telegram"
last_verified: "2026-06-04"
public: true
---

# Telegram Channel

`mortelos/channel-telegram` is the Telegram channel driver for MortelOS.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Inbound messages | Receive Telegram webhook calls and process inbound messages. |
| Outbound messages | Send messages through the Telegram Bot API. |
| Operations | Setup and health commands for the Telegram channel. |

## Install

```bash
composer require mortelos/channel-telegram
```

## Boundaries

The package owns the Telegram driver, API client, webhook route and inbound processing job. The host app owns bot credentials, tenant policy and which workflows can send or receive Telegram messages.

