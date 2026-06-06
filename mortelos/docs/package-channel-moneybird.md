---
title: "Moneybird Channel"
nav_title: "Moneybird channel"
slug: "package-channel-moneybird"
version: "0"
description: "Moneybird channel driver for scheduled invoice and contact sync."
section: "channels"
order: 60
status: "mvp"
audience: "developers"
package: "mortelos/channel-moneybird"
canonical_path: "/docs/0/package-channel-moneybird"
last_verified: "2026-06-04"
public: true
---

# Moneybird Channel

`mortelos/channel-moneybird` is the Moneybird channel driver for MortelOS.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Invoice sync | Synchronize invoices from Moneybird into MortelOS flows. |
| Contact sync | Synchronize contacts from Moneybird. |
| Revenue data | Provide Moneybird-backed revenue data requests. |
| Operations | Setup, sync and health commands for the Moneybird channel. |

## Install

```bash
composer require mortelos/channel-moneybird
```

## Boundaries

The package owns the Moneybird driver, API client, sync actions and setup provider. The host app owns credentials, sync schedule, tenant policy and how synced financial data appears in portal workflows.
