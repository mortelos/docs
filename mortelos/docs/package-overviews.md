---
title: "MortelOS Overviews"
nav_title: "Overviews"
slug: "package-overviews"
version: "0"
description: "Reusable flexible overview datasource, save-flow, chat widget and context integration for MortelOS."
section: "packages"
order: 55
status: "mvp"
audience: "developers"
package: "mortelos/overviews"
canonical_path: "/docs/0/package-overviews"
last_verified: "2026-10-16"
public: true
---

# MortelOS Overviews

`mortelos/overviews` provides reusable flexible overviews with datasource wiring, save-flow behavior, chat widgets and context integration.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Datasources | Overview datasource contracts, registry and entity-backed datasource support. |
| Query planning | Overview query planning and result shaping for reusable list surfaces. |
| Save flow | Save suggestions and overview saver behavior for package-owned overview actions. |
| Chat integration | Overview save prompt widget support through `mortelos/chat`. |
| Agent tools | Overview save and suggest-save tools when the agent tool registry is available. |

## Install

```bash
composer require mortelos/overviews
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Chat widget | `overview_save_prompt` |
| Agent tool | `overview.save` |
| Agent tool | `overview.suggest_save` |

## Upgrading to 0.4.0

Require `mortelos/framework:^0.7` and `mortelos/chat:^0.6`. Upgrade chat, widget-compliance and the host integrations together.

The browser event `overzicht-saved` is now `overview-saved`, for both dynamic and snapshot saves. There is no alias for the old event. Update every host listener, including Blade and Alpine event bindings, and rename handlers such as `onOverzichtSaved` to `onOverviewSaved`. Dutch customer-facing copy stays Dutch.

## Outlets

An outlet is a place where you extend `mortelos/overviews` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.3.1; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list overviews
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| contract | `Mortelos\Overviews\Support\OverviewDatasource` | Supplies the rows of a saved overview from one source (database, SQL, API). Default: `EntitiesOverviewDatasource`. |  |
| registry | `Mortelos\Overviews\Support\OverviewDatasourceRegistry` | Add an `OverviewDatasource`; an overview picks it with `attributes.datasource`. Register with `register()`. |  |
| config | `overviews` | Switches for the chat widget and the agent tool, and `overviews.entities.limit` (the maximum number of candidate rows; `null` is unlimited). Publish with `php artisan vendor:publish --tag=overviews-config`. |  |
| view | `overviews.save-prompt.actions` | Next to the Dynamic, Snapshot and Not now buttons in the save widget. Default: those three buttons. | `message` |
| view | `overviews.save-prompt.saved` | The confirmation after the overview is saved. Default: a check mark with a short Dutch confirmation. | `message` |

After saving, the save widget sends the Livewire browser event `overview-saved` with `overviewId`. It is not an outlet in the manifest, so it can change in a minor release.

## Boundaries

`mortelos/framework` owns entities, access and agent-run primitives. `mortelos/overviews` owns overview-specific product behavior. Keep one-off overview columns, tenant wording and local policy in the host until they become reusable.
