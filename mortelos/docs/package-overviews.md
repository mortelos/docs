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
last_verified: "2026-06-07"
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

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Chat widget | `overview_save_prompt` |
| Agent tool | `overview.save` |
| Agent tool | `overview.suggest_save` |

## Boundaries

`mortelos/framework` owns entities, access and agent-run primitives. `mortelos/overviews` owns overview-specific product behavior. Keep one-off overview columns, tenant wording and local policy in the host until they become reusable.
