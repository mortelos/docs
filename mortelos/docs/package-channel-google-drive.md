---
title: "Google Drive Channel"
nav_title: "Google Drive channel"
slug: "package-channel-google-drive"
version: "0"
description: "Google Drive channel driver for attachments, briefings and exported workspace material."
section: "channels"
order: 59
status: "mvp"
audience: "developers"
package: "mortelos/channel-google-drive"
canonical_path: "/docs/0/package-channel-google-drive"
last_verified: "2026-06-07"
public: true
---

# Google Drive Channel

`mortelos/channel-google-drive` is the Google Drive channel driver for MortelOS.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Drive push | Push attachments, briefings or exported workspace material to Google Drive. |
| Operations | Setup, push and health commands for the Drive channel. |
| Filesystem support | Google Drive filesystem integration through the package dependencies. |

## Install

```bash
composer require mortelos/channel-google-drive
```

## Boundaries

The package owns the Drive driver, API client and push job. The host app owns Google credentials, target folder policy, retention policy and which portal events should push material to Drive.
