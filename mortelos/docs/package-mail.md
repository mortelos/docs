---
title: "MortelOS Mail"
nav_title: "Mail"
slug: "package-mail"
version: "0"
description: "Reusable mail read models, indexing and search context for MortelOS."
section: "packages"
order: 57
status: "mvp"
audience: "developers"
package: "mortelos/mail"
canonical_path: "/docs/0/package-mail"
last_verified: "2026-06-08"
public: true
---

# MortelOS Mail

`mortelos/mail` provides reusable mail read models and indexing for MortelOS communication workflows.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Mail read models | Store normalized mail threads and messages from channel events. |
| Indexing | Prepare mail messages for search and embedding workflows. |
| Search context | Query actor-scoped mail context for OS communication and agent answers. |
| Channel projections | Listen to channel receive and send events and keep the mail read model current. |

## Install

```bash
composer require mortelos/mail
```

## Boundaries

Use this package for reusable mail storage, projections and search services. Channel-specific OAuth, polling and API clients stay in channel packages such as `mortelos/channel-gmail`. Host apps own tenant policy, credentials and local communication defaults.
