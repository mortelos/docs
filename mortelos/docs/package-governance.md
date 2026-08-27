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
last_verified: "2026-08-27"
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

Use `mortelos/dev-tools` when it is installed:

```bash
php artisan mortelos:package-decision "Customer Portal" \
  --decision=package-ready \
  --surface=mortelos/customer-portal \
  --reason="Reusable shell with customer-specific tenant policy and branding." \
  --no-interaction

php artisan mortelos:package-decisions:check --require-reason --no-interaction
```

When the dev tools are not installed, record the same fields in `.mortelos/package-decisions.md`:

```markdown
## Customer Portal

Surface: `mortelos/customer-portal`
Decision: `package-ready`
Reason: Reusable shell with customer-specific tenant policy and branding.
Date: 2026-06-07
```

CI should fail when package governance fails. In host apps, this is usually exposed through:

```bash
composer package-governance
```

## Local Source Layout

Every package has its own Git repository. Check them out as siblings in one workspace directory, one directory per package, and treat that checkout as the canonical local source for MortelOS work.

Host-local package folders such as `packages/mortelos/*` are only for short-lived testing. Before tagging or releasing a reusable package change, move the work into the matching package repository and tag that repository.

Host apps install every MortelOS package from the registry by version range. To try a local change against a host app, replace the installed copy with a symlink to your checkout after `composer install`:

```bash
rm -rf vendor/mortelos/mail
ln -s ~/Sites/mortelos/mail vendor/mortelos/mail
```

The symlink is local and uncommitted, and the next `composer install` removes it. Do not add a `path` repository to the committed `composer.json`: it pins the app to a machine-local directory and breaks CI and deploys.

## Agent Rules

Package-specific agent instructions should be generated from package rules and docs, then merged into the host `AGENTS.md`.

Host apps should regenerate the merged block during Composer updates:

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

When a package boundary, widget convention or agent tool contract is unclear, consult the MortelOS docs before implementation and update the relevant package docs when the convention changes.

## What stays in the host

Host apps own tenant config, local branding, policy defaults, local orchestration and customer-specific integrations.

Reusable packages own shared routes, views, Livewire namespaces, migrations, commands, extension contracts and tests when those concerns apply to more than one installation.

## Examples

| Capability | Likely decision | Reason |
| --- | --- | --- |
| Document review workflow | `package-ready` | The host may need local policy and branding first, but review workflow behavior is reusable. |
| Moneybird sync | `package-now` | The integration boundary can serve multiple installations. |
| One customer's internal KPI wording | `workspace-only` | The behavior is specific to that workspace. |
