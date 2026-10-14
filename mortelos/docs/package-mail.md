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
last_verified: "2026-10-14"
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

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Outlets

An outlet is a place where you extend `mortelos/mail` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.1.3; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list mail
```

| Kind | Name | What it is for |
| --- | --- | --- |
| config | `mortelos-mail` | The limit for the search text and the embedding settings (on or off, and on which queue) of the mail indexing. Publish with `php artisan vendor:publish --tag=mortelos-mail-config`. |

## Boundaries

Use this package for reusable mail storage, projections and search services. Channel-specific OAuth, polling and API clients stay in channel packages such as `mortelos/channel-gmail`. Host apps own tenant policy, credentials and local communication defaults.
