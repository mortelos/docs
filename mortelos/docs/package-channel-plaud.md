---
title: "Plaud Channel"
nav_title: "Plaud channel"
slug: "package-channel-plaud"
version: "0"
description: "Plaud channel driver for transcription ingestion."
section: "channels"
order: 61
status: "mvp"
audience: "developers"
package: "mortelos/channel-plaud"
canonical_path: "/docs/0/package-channel-plaud"
last_verified: "2026-10-14"
public: true
---

# Plaud Channel

`mortelos/channel-plaud` is the Plaud channel driver for MortelOS.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Plaud connection | Connect a MortelOS installation to Plaud. |
| Transcription submit | Submit recordings or source material for transcription. |
| Transcription ingest | Poll and store Plaud transcription results. |
| Formatting | Format Plaud transcription output for downstream channel flows. |

## Install

```bash
composer require mortelos/channel-plaud
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Outlets

An outlet is a place where you extend `mortelos/channel-plaud` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.4.1; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list channel-plaud
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| view | `channel-plaud.recording.panel` | A column next to the recording detail, only for its owner: process, share, chat. Default: nothing, the page stays one column. | `recordingId`, `recordingName` |
| config | `plaud-personal` | Connection to Plaud (`issuer`, `mcp_url`, `client_name`), routes (`route_prefix`, `middleware`, `return_route`) and sync (`sync_max_pages`, `timeout`). Publish with `php artisan vendor:publish --tag=plaud-personal-config`. |  |

The panel can dispatch the Livewire browser event `plaud-recording-audience-changed` to refresh the page. The connect routes check the gate ability `plaud-personal.connect`; define it in the host to decide who may connect a personal Plaud account. This browser event and gate are described here rather than as outlet rows in the manifest.

## Boundaries

The package owns the Plaud driver, API client, polling job and transcription formatting. The host app owns credentials, user consent, retention policy and local workflow routing.
