---
title: "MortelOS Framework"
nav_title: "Framework"
slug: "package-framework"
version: "0"
description: "Core MortelOS package for entities, links, events, projections, tenant primitives and runtime contracts."
section: "packages"
order: 52
status: "mvp"
audience: "developers"
package: "mortelos/framework"
canonical_path: "/docs/0/package-framework"
last_verified: "2026-06-07"
public: true
---

# MortelOS Framework

`mortelos/framework` is the core library for MortelOS primitives and runtime behavior.

The current PHP namespace can still be `Mortel\...` during the external package naming transition. Use `mortelos/framework` as the public package name in docs, plans and package decisions.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Entities and links | Entity registry, typed entity relationships and shared entity behavior. |
| Events and projections | Event-sourced facts, projection plumbing and reusable application services. |
| Tenant primitives | Tenant-aware context, access resolution and installation-level runtime contracts. |
| Agent runtime | MCP runtime, Laravel AI integration and safe workspace tool surfaces. |
| Document services | Shared framework services such as PDF, markdown and diff support where those belong below a feature package. |

## Install

```bash
composer require mortelos/framework
```

## Boundaries

Use this package for OS-level primitives that multiple MortelOS installations can share. Keep customer-specific policy defaults, branding, local orchestration and one-off integrations in the host app.

## Future Optimization Notes

When MortelOS optimization work moves from application-level cleanup to database-level tuning, consider PostgreSQL functions or procedures for small, data-heavy invariants. Good candidates are planner task ordering, append-to-day task planning, entity search ranking, memory recall candidate lookup, import-registry claims, AI action statistic increments and version publication counters.

Do not move broad workflows into the database by default. Keep external API calls, AI calls, embeddings, queue dispatch, Laravel events, mail delivery, policy orchestration and UI behavior in application code. Database routines should only own logic that benefits from atomicity, indexes, locking or reduced round trips.

## Runtime mount

Host apps mount the framework MCP server when operate mode is enabled:

```php
use Laravel\Mcp\Facades\Mcp;

Mcp::oauthRoutes();

Mcp::web('/mcp/mortelos', config('mortelos.mcp.server'))
    ->middleware(['auth:api']);
```

Production hosts add tenant initialization, role resolution, trust-level enforcement, data classification and throttling around that route.
