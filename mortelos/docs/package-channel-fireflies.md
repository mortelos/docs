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
last_verified: "2026-10-16"
public: true
---

# Fireflies Channel

`mortelos/channel-fireflies` is the Fireflies channel driver for MortelOS. Fireflies pushes meeting transcripts to a webhook. The package checks the signature and stores each transcript as a `transcript` entity, linked to a `meeting` entity. This page describes version 0.3.2.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Webhooks | Receive Fireflies webhook calls per tenant and check their HMAC-SHA256 signature. |
| Transcript ingest | Turn each new transcript into a `transcript` entity and a `meeting` entity, joined by a `has_transcript` link. |
| Classification | Mark both entities `confidential` when the payload looks finance-related, otherwise `internal`. |
| Operations | Setup and health commands for the Fireflies channel. |

## Install

```bash
composer require mortelos/channel-fireflies
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The package needs PHP 8.4 and `mortelos/framework` `^0.6.11` (0.6.11 or later within 0.6). The service provider is auto-discovered. It registers the `fireflies` driver with the framework's channel driver registry, loads the webhook route and adds two Artisan commands. A queued job processes webhook payloads, so the host needs a running queue worker.

## Set Up A Tenant

```bash
php artisan mortel:channel:fireflies:setup --tenant=<tenant-id-or-slug>
```

| Option | Meaning |
| --- | --- |
| `--tenant` | Tenant ID or slug. |
| `--api-key` | Fireflies API key. Without it, the command asks for the key. The key is required. |
| `--shared-secret` | Webhook signing secret. Without it, the command generates a random 48-character secret. |

The command creates one inbound channel named `Fireflies` for the tenant, or updates the existing one. It stores the API key and the shared secret encrypted on the channel and sets the channel to `connected`. Then it prints the webhook URL and the signing secret. Enter both in Fireflies:

```text
<APP_URL>/api/channels/fireflies/webhook?tenant=<tenant-slug>
```

Set `APP_URL` before you run the command. Without it, the command fails before it prints the webhook URL. The webhook finds the tenant by its `slug`, so give the tenant a slug first.

## Webhook

Fireflies sends the transcript payload to the webhook URL. The request must carry an `X-Fireflies-Signature` header with the hex HMAC-SHA256 of the raw request body, signed with the shared secret.

| Response | When |
| --- | --- |
| `200` `{"accepted": true}` | The signature is valid. The payload is queued for processing after the database commit. |
| `400` `missing_tenant` | The `tenant` query parameter is missing. |
| `404` `unknown_tenant` | No tenant has that slug. |
| `412` `channel_not_configured` | The tenant has no Fireflies channel, or no stored shared secret. |
| `401` `unauthorized` | The signature does not match. |

The webhook answers quickly and leaves the processing to the queued job.

## What Gets Stored

The job reads the transcript ID from `transcript_id`, `id` or `data.transcript_id`, in that order. A payload without a transcript ID is ignored.

Processing is idempotent. A transcript whose Fireflies ID is already stored is skipped. A cache lock per tenant and transcript stops two deliveries of the same transcript from running at the same time. The job tries three times, with a backoff of 60, 300 and 900 seconds.

| Entity | Attributes |
| --- | --- |
| `transcript` | `fireflies_transcript_id`, `meeting_url`, `participants`, `summary`, `body`, `classification`, `date` |
| `meeting` | `transcript_id` (the transcript entity), `fireflies_transcript_id`, `date`, `participants`, `classification` |

Both entities take the payload `title` as their name and have status `live`. They are written through the framework aggregates with the actor `channel:fireflies`. A `has_transcript` link runs from the meeting to the transcript.

Both entities are classified `confidential` when one of these is true. Otherwise they are `internal`.

- A tag contains `finance`, `confidential` or `vertrouwelijk`.
- A participant e-mail address starts with `finance@` or contains `@finance.`.
- The title starts with `[finance]` or `[confidential]`.

## Health Check

```bash
php artisan mortel:channel:fireflies:health --tenant=<tenant-id-or-slug>
```

The command checks the stored API key against the Fireflies API. It also counts all `transcript` entities created in the tenant in the last seven days, regardless of source.

| Exit code | Meaning |
| --- | --- |
| `0` | The API key works and at least one transcript came in during the last seven days. |
| `1` | The API key works, but no transcripts came in during the last seven days. |
| `2` | The tenant is unknown, the channel or its credentials are missing, or the API returns any non-2xx response, including 401 and 5xx. |

When Fireflies answers `401`, the command sets the channel to `disconnected`. It also sends a database notification to the tenant's users asking them to run the setup command again. The notification goes out only when the status changes, not on every check. Its message is in Dutch.

## Limits

- Inbound only. Sending through the driver returns the failure `outbound_not_supported`, and the driver does not poll Fireflies.
- Each tenant uses its newest Fireflies channel.
- Classification follows only the keyword rules above. It does not analyse the content.
- The API client calls `https://api.fireflies.ai/graphql`.

## Outlets

This package declares no outlets yet; it has no screens or seams a host plugs into.

## Boundaries

The package owns the Fireflies driver, API client, webhook route, processing job and the setup and health commands. The host app owns the Fireflies account, running the setup per tenant, the queue worker, tenant policy and what happens after transcript ingest.
