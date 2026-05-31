---
title: "Package Governance"
nav_title: "Package governance"
slug: "package-governance"
version: "0"
description: "Record package boundary decisions before adding MortelOS features."
section: "governance"
order: 40
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/package-governance"
last_verified: "2026-05-31"
public: true
---

# Package Governance

Every new feature, integration, frontend surface, backend service or reusable workflow starts with a package decision.

This keeps MortelOS composable. It also forces the team to decide what belongs in a reusable package and what belongs in the host app.

## Decisions

| Decision | Use when |
| --- | --- |
| `package-now` | The capability can serve another MortelOS installation today. Build directly in a package. |
| `package-ready` | The host needs local wiring first, but the package boundary is explicit. Extract when stable. |
| `workspace-only` | The behavior is specific to one customer or workspace. |

Default to `package-ready` when unsure. It preserves speed while keeping the future package boundary visible.

## Record a decision

```bash
php artisan mortelos:package-decision "Customer Portal" \
  --decision=package-ready \
  --surface=mortelos/customer-portal \
  --reason="Reusable shell with customer-specific tenant policy and branding." \
  --no-interaction

php artisan mortelos:package-decisions:check --require-reason --no-interaction
```

CI should fail when package governance fails.

```bash
composer package-governance
```

## What stays in the host

Host apps own tenant config, local branding, policy defaults, local orchestration and customer-specific integrations.

Reusable packages own shared routes, views, Livewire namespaces, migrations, commands, extension contracts and tests when those concerns apply to more than one installation.

