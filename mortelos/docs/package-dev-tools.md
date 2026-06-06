---
title: "MortelOS Dev Tools"
nav_title: "Dev tools"
slug: "package-dev-tools"
version: "0"
description: "Developer tooling for MortelOS package decisions and governance checks."
section: "packages"
order: 65
status: "mvp"
audience: "developers"
package: "mortelos/dev-tools"
canonical_path: "/docs/0/package-dev-tools"
last_verified: "2026-06-04"
public: true
---

# MortelOS Dev Tools

`mortelos/dev-tools` provides developer tooling for package-ready feature decisions.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Decision logging | Record whether a feature is `package-now`, `package-ready` or `workspace-only`. |
| Governance checks | Validate package decision logs locally or in CI. |
| Workflow support | Keep package boundaries explicit before implementation starts. |

## Install

```bash
composer require mortelos/dev-tools --dev
```

## Commands

```bash
php artisan mortelos:package-decision "Customer Portal" \
  --decision=package-ready \
  --surface=mortelos/customer-portal \
  --reason="Reusable shell with customer-specific tenant policy and branding."

php artisan mortelos:package-decisions:check --require-reason
```

## Boundaries

Use this package for development workflow and governance. It does not own runtime portal behavior.

