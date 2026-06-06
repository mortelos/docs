---
title: "Plaud Channel"
nav_title: "Plaud channel"
slug: "package-channel-plaud"
version: "0"
description: "Plaud channel driver for transcription ingestion."
section: "packages"
order: 61
status: "mvp"
audience: "developers"
package: "mortelos/channel-plaud"
canonical_path: "/docs/0/package-channel-plaud"
last_verified: "2026-06-04"
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

## Boundaries

The package owns the Plaud driver, API client, polling job and transcription formatting. The host app owns credentials, user consent, retention policy and local workflow routing.

