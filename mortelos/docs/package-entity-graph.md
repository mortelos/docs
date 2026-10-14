---
title: "MortelOS Entity Graph"
nav_title: "Entity graph"
slug: "package-entity-graph"
version: "0"
description: "Reusable entity graph traversal, API and visualization package for MortelOS."
section: "packages"
order: 56
status: "mvp"
audience: "developers"
package: "mortelos/entity-graph"
canonical_path: "/docs/0/package-entity-graph"
last_verified: "2026-10-14"
public: true
---

# MortelOS Entity Graph

`mortelos/entity-graph` provides entity graph traversal, API routes and visualization for MortelOS installations.

## Use It For

| Surface | Purpose |
| --- | --- |
| `GET /api/v1/entity-graph` | Build a capped graph around one entity. |
| `GET /api/v1/entity-graph/expand` | Load a small expansion graph around one node. |
| `GET /api/v1/entity-graph/search` | Search entities for graph focus and targeting. |
| `GET /api/v1/entity-graph/path` | Find a shortest visible path between two entities. |
| `entity-graph::panel` | Livewire graph workspace with canvas rendering, search, filters, inspector, audit and path-finding. |
| `entity_graph` chat widget | Render the graph workspace inside chat for a visible entity. |
| `entity.graph` agent tool | Resolve a visible entity and open the graph chat widget. |

## Install

```bash
composer require mortelos/entity-graph
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## V2 graph contract

Graph responses remain compatible with the v1 `nodes`, `edges`, `facets` and `meta` structure. V2 adds:

| Field | Purpose |
| --- | --- |
| `meta.schema_version` | Current graph response schema, currently `2`. |
| `meta.snapshot_id` | Stable hash for the visible graph and active filters. |
| `meta.query_ms` | Backend graph build time in milliseconds. |
| `meta.truncation_reason` | `max_nodes` or `max_edges` when caps are hit. |
| `attributes.metrics` | Node, edge, type and relation summary. |
| `attributes.top_connectors` | Highest degree visible nodes for hub analysis. |
| `attributes.relation_type_counts` | Flat relation count map for fast UI legends. |

The default UI renderer is `canvas` through `entity-graph.ui.renderer`. Set it to another value to use the legacy SVG renderer as rollback.

## Outlets

An outlet is a place where you extend `mortelos/entity-graph` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.5.0; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list entity-graph
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| contract | `Mortelos\EntityGraph\Contracts\EntityGraphAccessContextResolver` | Decides for which organisation, branch and role a graph is built. Default: `RequestEntityGraphAccessContextResolver`. |  |
| contract | `Mortelos\EntityGraph\Contracts\EntityGraphAuditTrailResolver` | Supplies the events for the Audit tab in the inspector. Default: `DefaultEntityGraphAuditTrailResolver`. |  |
| contract | `Mortelos\EntityGraph\Contracts\EntityGraphNodePresenter` | Adds a label, colour, cluster or link to each node before it goes to the browser. Default: `DefaultEntityGraphNodePresenter`. |  |
| config | `entity-graph` | Routes and middleware (`api`, `runtime`), limits, profiles, labels, `relation_types` and `collapse_leaf_relations`. A group you publish replaces the whole group; the `middleware` and `policy_middleware` keys of `api` and `runtime` then fall back to the safe package default, never to "no check". Set `policy_middleware` to `null` to switch the policy check off on purpose. Publish with `php artisan vendor:publish --tag=entity-graph-config`. |  |
| view | `entity-graph.panel.header-actions` | A row of actions above the graph panel, aligned right. The panel itself sits inside `wire:ignore`; this place is outside it. The row only shows when something is plugged in. Default: nothing. | `entity`, `scope`, `profile` |

A host can also embed the Livewire component `entity-graph::panel` or the Blade partial `entity-graph::partials.graph-canvas` (a fixed graph you pass in yourself), which use the browser script `resources/js/runtime.js`. These are not outlets in the manifest, so they can change in a minor release.

## Boundaries

Use this package for reusable entity graph behavior. Bind the contract outlets in the host app when a customer needs custom access context, audit details or node presentation.
