---
title: "MortelOS Agent Standards"
nav_title: "Agent standards"
slug: "package-agent-standards"
version: "0"
description: "MortelOS-owned AI agent instructions for host apps and package-aware development."
section: "tooling"
order: 66
status: "mvp"
audience: "developers"
package: "mortelos/agent-standards"
canonical_path: "/docs/0/package-agent-standards"
last_verified: "2026-08-27"
public: true
---

# MortelOS Agent Standards

`mortelos/agent-standards` carries reusable AI agent instructions for MortelOS host apps.

This package is a MortelOS-owned hard-fork-style standard. Spatie's AI guidelines and Laravel community conventions are useful references, but MortelOS owns the actual rules that agents receive in `AGENTS.md`.

## Why It Exists

Agent instructions drift when every host app edits its own long prompt. Drift makes agents inconsistent and makes reviews harder.

The standard is packaged so every host app can install one reusable source of truth, then merge it into the root `AGENTS.md` with `mortelos/dev-tools`.

## Install In A Host App

```bash
composer require mortelos/agent-standards --dev
composer require mortelos/dev-tools --dev
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

Host apps should publish package rules after Composer updates:

```json
{
  "scripts": {
    "post-autoload-dump": [
      "@php artisan package:discover --ansi",
      "@php artisan mortelos:agent-rules:publish --target=AGENTS.md --no-interaction"
    ]
  }
}
```

Then publish or check manually when needed:

```bash
php artisan mortelos:agent-rules:publish --target=AGENTS.md --no-interaction
php artisan mortelos:agent-rules:check --target=AGENTS.md --no-interaction
```

## What It Covers

| Area | Rule family |
| --- | --- |
| Working method | Read repo instructions, keep work scoped, verify before reporting completion. |
| Laravel and PHP | Strict types, explicit domain boundaries, immutable dates and strict models. |
| TALL and UI | Use local Livewire and Flux conventions, keep components presentational. |
| Security | Validate and authorize at boundaries, guard tenant scope, fake external calls in tests. |
| Version control | Keep changes focused and respect dirty worktrees. |
| Dutch copy | Plain Dutch prose and no em dashes in Dutch user-facing text. |

## Host-Specific Rules

Keep tenant-specific instructions outside the generated block in `AGENTS.md`.

The generated block is owned by `mortelos/dev-tools` and overwritten by:

```bash
php artisan mortelos:agent-rules:publish
```

Use local host prose only for details that cannot live in a reusable package. If a rule should apply across all OS apps, move it into `mortelos/agent-standards`.

## Boundaries

`mortelos/agent-standards` is only instruction content. It does not add runtime behavior. Pair it with `mortelos/dev-tools` for publish/check commands, and pair host apps with `mortelos/app-standards` for Laravel runtime defaults.
