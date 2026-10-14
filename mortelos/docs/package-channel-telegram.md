---
title: "Telegram Channel"
nav_title: "Telegram channel"
slug: "package-channel-telegram"
version: "0"
description: "Telegram channel driver for bidirectional Bot API communication."
section: "channels"
order: 62
status: "mvp"
audience: "developers"
package: "mortelos/channel-telegram"
canonical_path: "/docs/0/package-channel-telegram"
last_verified: "2026-10-14"
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

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Outlets

An outlet is a place where you extend `mortelos/channel-telegram` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.3.2; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list channel-telegram
```

| Kind | Name | What it is for |
| --- | --- | --- |
| event | `Uteq\TelegramChannel\Events\TelegramCallbackReceived` | A user clicks an inline button inside the tenant context; the host interprets the callback data. |
| event | `Uteq\TelegramChannel\Events\TelegramMessageReceived` | An inbound message has been stored as an inbox item inside the tenant context. |

## Boundaries

The package owns the Telegram driver, API client, webhook route and inbound processing job. The host app owns bot credentials, tenant policy and which workflows can send or receive Telegram messages.
