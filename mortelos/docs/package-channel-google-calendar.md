---
title: "Google Calendar Channel"
nav_title: "Google Calendar channel"
slug: "package-channel-google-calendar"
version: "0"
description: "Google Calendar channel driver for reading calendar events with per-user Google OAuth."
section: "channels"
order: 63
status: "mvp"
audience: "developers"
package: "mortelos/channel-google-calendar"
canonical_path: "/docs/0/package-channel-google-calendar"
last_verified: "2026-09-26"
public: true
---

# Google Calendar Channel

`mortelos/channel-google-calendar` is the Google Calendar channel driver for MortelOS.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Inbound events | Read up to 50 upcoming events from a calendar (`primary` unless the `calendar_id` option is set) as inbound channel payloads. The driver is inbound-only and rejects outbound sends. |
| Connector setup | Start Google OAuth for a personal channel owned by the acting user, with the `calendar.events` scope and offline access. Running setup again reuses that channel and issues a fresh authorization URL. |
| API client | List upcoming events or events in a time range, insert and update events, and refresh an expired access token. |
| Scheduled polling | Add a poll command to the Laravel schedule that queues a poll job every five minutes for each connected or degraded Google Calendar channel. Each job refreshes an expired access token and reads upcoming events. |

## Install

```bash
composer require mortelos/channel-google-calendar
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Channel driver | `google_calendar` |
| Connector setup provider | `google_calendar` |

## Boundaries

The package owns the Google Calendar driver, API client, connector setup provider, poll command and poll job. `mortelos/framework` owns the channel model and the driver and connector setup registries the package registers into.

The host app owns the Google OAuth client settings (`services.google.client_id`, `services.google.client_secret`, `services.google.redirect`), the OAuth callback behind that redirect URI, and running the scheduler. The package ships no callback route: the callback must store the tokens as the channel's encrypted `credentials_reference` and mark the channel connected before polling picks it up. The poll job does not store the events it reads or the refreshed token, so syncing events into local records and writing events back to Google belong to the consuming app or package.
