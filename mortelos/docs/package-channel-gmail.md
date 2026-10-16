---
title: "Gmail Channel"
nav_title: "Gmail channel"
slug: "package-channel-gmail"
version: "0"
description: "Gmail channel driver for inbound and outbound workspace mail."
section: "channels"
order: 58
status: "mvp"
audience: "developers"
package: "mortelos/channel-gmail"
canonical_path: "/docs/0/package-channel-gmail"
last_verified: "2026-10-16"
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
| Connector setup | Register the Gmail connector provider so chat can start Google OAuth. |

## Install

```bash
composer require mortelos/channel-gmail
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Upgrading to 0.6.0

This release is a minor bump because public attachment names changed. Update the package constraint in your app to `^0.6`. The framework constraint is unchanged.

| 0.5 | 0.6 |
| --- | --- |
| named argument `bijlagen` in `GmailApiClient::createDraft()` and `updateDraft()` | `attachments` |
| attachment array key `naam` | `name` |
| attachment array key `inhoud` | `body` |

The attachment key `mime` is unchanged. Update any stored attachment arrays before you pass them to the client.

## Outlets

An outlet is a place where you extend `mortelos/channel-gmail` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.5.0; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list channel-gmail
```

| Kind | Name | What it is for |
| --- | --- | --- |
| config | `gmail-channel` | Turns fetching unread mail on or off (`poll_inbound`, off by default). Publish with `php artisan vendor:publish --tag=gmail-channel-config`. |
| event | `Mortel\Events\Channel\MailReceived` | After each fetched mail, before classification turns it into inbox items. |
| event | `Mortel\Events\Channel\MailSent` | After each mail sent through Gmail. |
| contract | `Mortel\Contracts\OAuthCredentialsResolver` | Supplies the client ID, client secret and redirect URI of the Google OAuth app (provider `google`); by default from `services.google.*`. Default: `ConfigOAuthCredentialsResolver`. |

## Boundaries

The package owns the Gmail driver, API client, polling jobs, send-mail action and connector setup provider. The host app owns Google credentials, tenant policy, message retention choices and local workflow decisions.
