---
title: "MortelOS UI"
nav_title: "UI"
slug: "package-ui"
version: "0"
description: "Shared UI primitives consumed by MortelOS packages and host apps."
section: "packages"
order: 53
status: "mvp"
audience: "developers"
package: "mortelos/ui"
canonical_path: "/docs/0/package-ui"
last_verified: "2026-08-27"
public: true
---

# MortelOS UI

`mortelos/ui` provides shared interface primitives for MortelOS host apps and reusable packages.

## Use It For

| Area | Package responsibility |
| --- | --- |
| UI primitives | Shared Flux-aligned Blade and Livewire interface building blocks. |
| Package surfaces | Reusable components that packages can consume without copying host app UI. |
| Design consistency | Common visual primitives for starter screens and package-owned views. |

## Install

```bash
composer require mortelos/ui
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Boundaries

Use this package for reusable interface primitives. Keep customer branding, page composition, local copy and domain-specific workflows in the host app or in the package that owns that domain surface.
