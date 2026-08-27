---
title: "MortelOS Dev Tools"
nav_title: "Dev tools"
slug: "package-dev-tools"
version: "0"
description: "Developer tooling for MortelOS package decisions and governance checks."
section: "tooling"
order: 65
status: "mvp"
audience: "developers"
package: "mortelos/dev-tools"
canonical_path: "/docs/0/package-dev-tools"
last_verified: "2026-08-27"
public: true
---

# MortelOS Dev Tools

`mortelos/dev-tools` provides developer tooling for package-ready feature decisions.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Decision logging | Record whether a feature is `package-now`, `package-ready` or `workspace-only`. |
| Governance checks | Validate package decision logs locally or in CI. |
| Agent rules | Publish and check merged agent rules for package-aware development, including `mortelos/agent-standards`. |
| Workflow support | Keep package boundaries explicit before implementation starts. |

## Install

```bash
composer require mortelos/dev-tools --dev
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

Install `mortelos/agent-standards` alongside it when the host should receive the shared MortelOS AI rules:

```bash
composer require mortelos/agent-standards --dev
```

## Commands

```bash
php artisan mortelos:package-decision "Customer Portal" \
  --decision=package-ready \
  --surface=mortelos/customer-portal \
  --reason="Reusable shell with customer-specific tenant policy and branding."

php artisan mortelos:package-decisions:check --require-reason
php artisan mortelos:agent-rules:publish
php artisan mortelos:agent-rules:check
```

## Composer Hook

Host apps should regenerate package agent rules after Composer updates, similar to Laravel Boost package discovery hooks:

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

Keep host-specific instructions outside the generated block. Manual edits inside the generated block are overwritten.

## Boundaries

Use this package for development workflow and governance. It does not own runtime portal behavior.
