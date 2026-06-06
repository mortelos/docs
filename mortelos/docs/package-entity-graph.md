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
last_verified: "2026-06-04"
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

## Boundaries

Use this package for reusable entity graph behavior. Bind the extension contracts in the host app when a customer needs custom access context, audit details or node presentation.

