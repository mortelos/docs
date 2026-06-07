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
last_verified: "2026-06-07"
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

## Extension Contracts

| Contract | Purpose |
| --- | --- |
| `EntityGraphAccessContextResolver` | Resolves trust, org and branch context for API and UI calls. |
| `EntityGraphAuditTrailResolver` | Supplies the inspector Audit tab. |
| `EntityGraphNodePresenter` | Adds labels, colors, clusters and detail URLs to nodes. |

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

## Boundaries

Use this package for reusable entity graph behavior. Bind the extension contracts in the host app when a customer needs custom access context, audit details or node presentation.
