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
last_verified: "2026-10-16"
public: true
---

# MortelOS Agent Standards

`mortelos/agent-standards` carries reusable AI agent instructions for MortelOS host apps and packages. This page describes version 0.1.15.

This package is a MortelOS-owned hard-fork-style standard. Spatie's AI guidelines and Laravel community conventions are useful references, but MortelOS owns the actual rules that agents receive in `AGENTS.md`.

## Why It Exists

Agent instructions drift when every host app edits its own long prompt. Drift makes agents inconsistent and makes reviews harder.

The standard is packaged so every host app can install one reusable source of truth, then merge it into the root `AGENTS.md` with `mortelos/dev-tools`. The rules live in the package at `resources/mortelos/agent-rules/AGENTS.md`.

## Install In A Host App

```bash
composer require mortelos/agent-standards --dev
composer require mortelos/dev-tools --dev
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The package needs PHP 8.4 and `illuminate/support` 13. The service provider is auto-discovered.

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

To keep a plain copy of the rules in the host, publish them with the package tag. This writes `resources/mortelos/agent-rules/agent-standards.md`:

```bash
php artisan vendor:publish --tag=mortelos-agent-standards-rules
```

## Loading The Rules In Claude Code

Claude Code reads `CLAUDE.md`, not `AGENTS.md`, when both exist.

- In a host app, keep the generated block in `AGENTS.md` and put `@AGENTS.md` in `CLAUDE.md`.
- In a package, require `mortelos/agent-standards` as a dev dependency from the MortelOS registry, not as a symlinked path repository. Then put this line in the package `CLAUDE.md`:

```text
@vendor/mortelos/agent-standards/resources/mortelos/agent-rules/AGENTS.md
```

An agent that works in a package from a parent directory loads that `CLAUDE.md` when it opens a file in the package with the Read tool.

## What It Covers

| Area | Rule family |
| --- | --- |
| Working method | Read repo instructions, keep work scoped, verify before reporting completion. |
| Laravel and PHP | Strict types, explicit domain boundaries, immutable dates and strict models. |
| TALL and UI | Use Livewire 4 single-file components and `x-mortel::` components from `mortelos/ui` first. Use Flux (`flux:`) only when `mortelos/ui` has no matching component; never write custom HTML for something it already provides. Keep components presentational. |
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

## Code Language

Since 0.1.15 the rules say that all code is English. Dutch is for text a customer sees, with a short list of exceptions. `CodeLanguageGuard` in `mortelos/dev-tools` enforces the rule. See [Coding guidelines](/docs/0/coding-guidelines).

## Outlets

This package declares no outlets yet; it has no screens or seams a host plugs into.

## Boundaries

`mortelos/agent-standards` is only instruction content. It adds no runtime behaviour: its service provider only registers the publish tag, and only in the console. Pair it with `mortelos/dev-tools` for publish and check commands, and pair host apps with `mortelos/app-standards` for Laravel runtime defaults.
