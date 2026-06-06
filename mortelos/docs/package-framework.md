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
last_verified: "2026-06-04"
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

