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
last_verified: "2026-10-14"
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

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Outlets

An outlet is a place where you extend `mortelos/channel-moneybird` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.4.6; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list channel-moneybird
```

| Kind | Name | What it is for |
| --- | --- | --- |
| config | `channel-moneybird` | Which entity types the Moneybird sync writes to: `contact_type`, `invoice_type` and `invoice_contact_relation`. Publish with `php artisan vendor:publish --tag=channel-moneybird-config`. |

## Boundaries

The package owns the Moneybird driver, API client, sync actions and setup provider. The host app owns credentials, sync schedule, tenant policy and how synced financial data appears in portal workflows.
