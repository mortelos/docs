---
title: "Pocket Channel"
nav_title: "Pocket channel"
slug: "package-channel-pocket"
version: "0"
description: "Pocket channel driver that ingests recordings as meeting entities with transcript, summary and action items."
section: "channels"
order: 64
status: "mvp"
audience: "developers"
package: "mortelos/channel-pocket"
canonical_path: "/docs/0/package-channel-pocket"
last_verified: "2026-10-16"
public: true
---

# Pocket Channel

`mortelos/channel-pocket` is the Pocket channel driver for MortelOS: it turns conversation recordings from Pocket (heypocket.com) into `meeting` entities with transcript, summary and action items.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Pocket connection | Check a Pocket API key and store it encrypted on an inbound `pocket` channel per tenant. |
| Recording ingest | Poll the Pocket public API and fetch details only for completed recordings that are new or changed since the last run. |
| Meetings | Create or update one `meeting` entity per recording, keyed on `pocket_recording_id`. Fields Pocket does not supply, such as `notes` or an edited `status`, are kept on update. |
| Meeting schema | Create the `meeting` entity type when it is missing, or add the Pocket fields to an existing one. |
| Channel driver | Register an inbound-only `pocket` driver. Sending through Pocket is not supported. |

## Install

```bash
composer require mortelos/channel-pocket
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Commands

```bash
php artisan channel:connect-pocket --tenant=example
php artisan channel:poll-pocket
```

`channel:connect-pocket` reads the API key from the host's `services.pocket.api_key` config value (for example from a `POCKET_API_KEY` environment variable), never from the command line. It checks the key against Pocket, creates the tenant's `pocket` channel if there is none (`--name` sets its name, default `Pocket`), stores the key encrypted and marks the channel connected.

`channel:poll-pocket` polls every tenant with a connected or degraded `pocket` channel, or only the tenant passed with `--tenant` (ID or slug). A failure for one tenant does not stop the others, but the command then exits with a failure code. The package does not schedule this command; add it to the host's scheduler.

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Channel driver | `pocket` |
| Entity type | `meeting` |
| Meeting fields added | `pocket_recording_id`, `pocket_updated_at`, `transcript`, `action_items`, `calendar_event_id` |

## Upgrading to 0.2.0

This release is a minor bump because public return values and a stored status changed. Update the package constraint in your app to `^0.2`. The framework constraint and the Dutch command output are unchanged.

| 0.1 | 0.2 |
| --- | --- |
| `StorePocketRecording::handle()` returns `nieuw` | `created` |
| `StorePocketRecording::handle()` returns `bijgewerkt` | `updated` |
| `StorePocketRecording::handle()` returns `ongewijzigd` | `unchanged` |
| status `attributes.status = opgenomen` on a new meeting | `recorded` |

Update callers that compare or count the results. Migrate existing meeting statuses from `opgenomen` to `recorded`, and update application filters and renderers. Other statuses that a user edited are kept.

If you use the default meeting layout of `mortelos/record`, upgrade it to `^0.3` together with this package. That layout now recognises `recorded`.

## Outlets

This package declares no outlets yet; it has no screens or seams a host plugs into.

## Boundaries

The package owns the Pocket driver, API client, connect and poll commands and the mapping from a Pocket recording to a `meeting` entity. Ingest is poll-based and read-only: the package registers no routes or webhooks and never writes to Pocket. The host app owns the Pocket API key, the polling schedule, recording consent, retention policy and what happens with meetings after ingest.
