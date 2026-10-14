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
last_verified: "2026-10-14"
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

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Boundaries

Use this package for OS-level primitives that multiple MortelOS installations can share. Keep customer-specific policy defaults, branding, local orchestration and one-off integrations in the host app.

## Action results, undo and actors

Since v0.6.38 every write goes through the same few types. The starter shows them in use: `app/Actions/Invitations/InviteUser.php` and `app/Providers/StarterServiceProvider.php`.

- `Mortel\Actions\ActionResult` is what an Action returns. Its status is `done`, `approval_needed`, `failed` or `skipped`. A refusal by policy stays an exception.
- `Mortel\Actions\Undo` is the recipe to reverse a change: a counter-Action class, its params and a `validUntil`. It lives in the result, not in the event.
- `Mortel\Actions\UndoRegistry` maps a counter-Action to its handler. The counter-Action records its own event, and that event is the audit.
- `Mortel\Support\Actor` says who acts, in four forms: `Actor::user()` (the bare user id), `Actor::agent()` (`agent:<run-id>`), `Actor::system()` (`system:<process>`) and `Actor::bridge()` (`bridge:<name>`).
- `EventContext::within($actor, $reason, $work)` fills actor, reason, org and branch around the work and restores the previous context afterwards.
- `PolicyScope::Action` is the policy scope of host Action abilities such as `invitations.manage`.
- `config('mortel.events.schema_namespaces')` opts a host namespace in to payload validation. The schema file sits next to the event class, and its name follows the class name: `InvitationCreated` is validated against `invitation.created.v1.schema.json`.
- `handle()` is the method on every Action. `execute()` still works on the Actions that had it.
- The `events.actor` column is 255 characters wide from v0.6.41, so `agent:`, `system:` and `bridge:` actors fit.

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
