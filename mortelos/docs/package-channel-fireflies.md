---
title: "Fireflies Channel"
nav_title: "Fireflies channel"
slug: "package-channel-fireflies"
version: "0"
description: "Fireflies channel driver for inbound webhooks and transcript ingest."
section: "channels"
order: 57
status: "mvp"
audience: "developers"
package: "mortelos/channel-fireflies"
canonical_path: "/docs/0/package-channel-fireflies"
last_verified: "2026-06-07"
public: true
---

# Fireflies Channel

`mortelos/channel-fireflies` is the Fireflies channel driver for MortelOS.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Webhooks | Receive Fireflies inbound webhook calls. |
| Transcript ingest | Process transcript payloads into MortelOS channel flows. |
| Operations | Setup and health commands for the Fireflies channel. |

## Install

```bash
composer require mortelos/channel-fireflies
```

## Boundaries

The package owns the Fireflies driver, API client, webhook route and processing job. The host app owns credentials, tenant policy and what happens after transcript ingest.
