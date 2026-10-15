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
last_verified: "2026-10-14"
public: true
---

# MortelOS Dev Tools

`mortelos/dev-tools` provides developer tooling for package-ready feature decisions.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Decision logging | Record whether a feature is `package-now`, `package-ready`, `workspace-only` or `build-in-mortelos`. |
| Governance checks | Validate package decision logs locally or in CI. |
| Agent rules | Publish and check merged agent rules for package-aware development, including `mortelos/agent-standards`. |
| Workflow support | Keep package boundaries explicit before implementation starts. |
| Outlets | List the outlets of the installed packages, check a package's `outlets.json` against its code, and fail a host build that works around a package. |

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
php artisan mortelos:outlets:list [package] [--kind=view] [--json]
php artisan mortelos:outlets:check
```

`mortelos:outlets:list` shows what you can plug into: package, kind, name, description and default. `mortelos:outlets:check` runs the same scan as `OutletsGuard` below, without an allow list; it exits with 1 on a violation and with 0 when there are only warnings. Both commands ship with 0.1.7.

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

## Outlet checks

Two test helpers keep outlets honest. Both ship with 0.1.7; [Outlets](/docs/0/outlets) explains what an outlet is.

`OutletsManifest` is for package authors. One test in the package keeps `resources/mortelos/outlets.json` in line with the code, in a test folder that `phpunit.xml` runs:

```php
use Mortelos\DevTools\Testing\OutletsManifest;

it('keeps outlets.json in line with the code', fn () => OutletsManifest::assertValid(dirname(__DIR__, 2)));
```

`OutletsGuard` is for host apps. One test next to `ArchGuardTest` fails the build when the app works around a package instead of using its outlets:

```php
use Mortelos\DevTools\Testing\OutletsGuard;

it('extends mortelos packages only through their outlets', function () {
    OutletsGuard::assertClean(base_path());
});
```

It checks seven rules: `unknown-outlet`, `concrete-subclass`, `config-set`, `vendor-require`, `vendor-patch`, `vendor-view-copy` and `manifest`. The table on the [Outlets](/docs/0/outlets#how-the-check-runs) page says what each one catches. It works like the other guard in this package, `ArchGuard`: an existing app starts with an allow list of counts per file that may only go down, and `outlets-guard: ignore <reason>` on a PHP line, or on the line above it, skips that PHP line.

## Outlets

This package declares no outlets yet; it has no screens or seams a host plugs into.

## Boundaries

Use this package for development workflow and governance. It does not own runtime portal behavior.
