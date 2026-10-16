---
title: "MortelOS App Standards"
nav_title: "App standards"
slug: "package-app-standards"
version: "0"
description: "MortelOS-owned Laravel runtime defaults for host applications."
section: "packages"
order: 54
status: "mvp"
audience: "developers"
package: "mortelos/app-standards"
canonical_path: "/docs/0/package-app-standards"
last_verified: "2026-10-16"
public: true
---

# MortelOS App Standards

`mortelos/app-standards` owns the Laravel runtime defaults every MortelOS host app should start with. It also carries the Spine contract: the governance rules every MortelOS portal inherits. This page describes version 0.1.3.

This package is intentionally a MortelOS-owned hard-fork-style reimplementation, not a runtime dependency on `nunomaduro/essentials`. Upstream packages and community conventions can inspire defaults, but MortelOS owns the behavior, names, tests and release cadence.

## Why It Exists

App-wide Laravel defaults are powerful. They affect Eloquent, validation, URLs, console safety and test behavior. Putting those defaults in a small MortelOS package gives every host app the same baseline without making reusable feature packages carry app-level side effects.

Use this package in host apps such as `mortelos/starter` and other concrete OS installations. Do not require it from reusable packages such as `mortelos/framework`, channels, widgets or studio packages.

## Defaults

| Area | Default |
| --- | --- |
| Eloquent | Strict models are enabled. |
| Relationships | Automatic relationship eager loading is enabled. |
| Mass assignment | Models stay guarded. Unguarding is available but off. |
| Dates | Laravel's date factory returns immutable dates. |
| URLs | HTTPS URL generation is forced in production. |
| Console safety | Destructive migration and database commands are prohibited in production. |
| Tests | Stray HTTP requests are prevented in testing. |
| Tests | `Sleep` is faked in testing. |
| Validation | Password defaults use a minimum length of 12, max length of 255 and uncompromised checks. |

Console safety is production-only on purpose. Test suites and local setup commands still need migrations such as `migrate:fresh` and `RefreshDatabase`.

## Install In A Host App

```bash
composer require mortelos/app-standards
```

The package resolves from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The package needs PHP 8.4 and Laravel 13. The service provider is auto-discovered.

To test a local checkout of the package against a host app, symlink it into `vendor/` after installing. Do not add a `path` repository to the committed `composer.json`; see [Packages](/docs/0/packages).

## Configuration

Publish the config when a host needs to override a default:

```bash
php artisan vendor:publish --tag=mortelos-app-standards-config
```

The published file is `config/app-standards.php`. Each switch also has an environment variable:

| Key | Environment variable | Default |
| --- | --- | --- |
| `strict_models` | `MORTELOS_STANDARDS_STRICT_MODELS` | `true` |
| `auto_eager_loading` | `MORTELOS_STANDARDS_AUTO_EAGER_LOADING` | `true` |
| `unguard_models` | `MORTELOS_STANDARDS_UNGUARD_MODELS` | `false` |
| `immutable_dates` | `MORTELOS_STANDARDS_IMMUTABLE_DATES` | `true` |
| `force_https.enabled` | `MORTELOS_STANDARDS_FORCE_HTTPS` | `true`, in `production` |
| `safe_console.enabled` | `MORTELOS_STANDARDS_SAFE_CONSOLE` | `true`, in `production` |
| `prevent_stray_requests.enabled` | `MORTELOS_STANDARDS_PREVENT_STRAY_REQUESTS` | `true`, in `testing` |
| `fake_sleep.enabled` | `MORTELOS_STANDARDS_FAKE_SLEEP` | `true`, in `testing` |
| `passwords.enabled` | `MORTELOS_STANDARDS_PASSWORDS` | `true` |

The environment-bound defaults have an `environments` list in the config file. An empty list applies the default in every environment. With `fake_sleep.sync_with_carbon` (default `false`), faked sleeps also move Carbon's test clock. The password rule reads `passwords.min`, `passwords.max`, `passwords.uncompromised` and `passwords.compromised_threshold`.

Prefer environment-specific toggles over removing the package. If a default is wrong for most MortelOS hosts, change it in this package and cover the behavior with a package test.

## Spine Contract

The Spine is the governance default every MortelOS portal inherits:

| Clause | Rule |
| --- | --- |
| C1 | Reading in-scope data is direct. No proposal, no approval. |
| C2 | Every mutation of governed state goes through a human-approved proposal. |
| C3 | Access is deny-by-default across entities, links, tools and surfaces. |
| C4 | The audit trail is immutable and event-sourced. |

Publish the contract and its config with:

```bash
php artisan vendor:publish --tag=mortelos-spine-contract
```

This writes `config/spine.php` and `.mortelos/spine-contract.md`. The config holds:

| Key | What it holds |
| --- | --- |
| `governed_models` | Framework models whose rows are governed state: entities, entity links, policies, roles, inbox items, channels, workflows, workflow runs, documents and document versions. |
| `writers` | `fnmatch()` patterns of classes that may write governed models directly. The defaults are the framework and `App\` aggregates and projectors. |
| `writer_attributes` | Attributes that mark a sanctioned writer. The default is `Mortelos\AppStandards\Spine\GovernedWriter`. |

Tag a governed action that does not fit the namespace patterns with the attribute. It takes an optional `reason` and changes no runtime behaviour:

```php
use Mortelos\AppStandards\Spine\GovernedWriter;

#[GovernedWriter(reason: 'Applies an approved proposal')]
final class ApplyApprovedChange
{
    // ...
}
```

`mortelos/dev-tools` checks the contract. `php artisan mortelos:spine:check` proves the rails exist and are connected. Its PHPStan rule flags direct writes to governed models from other classes; `@spine-ignore` on the line silences a deliberate exception. Include the rule in `phpstan.neon`:

```neon
includes:
    - vendor/mortelos/dev-tools/extension.neon
```

Neither check proves that a mutation really went through a proposal a human approved. Keep a test in every portal for its proposal path: propose, pending, human approves, state changes, event recorded.

## Outlets

An outlet is a place where you extend `mortelos/app-standards` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.1.3; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list app-standards
```

| Kind | Name | What it is for |
| --- | --- | --- |
| config | `spine` | Configure the governed models and permitted writers used by the spine checks. Publish with `php artisan vendor:publish --tag=mortelos-spine-contract`. |
| config | `app-standards` | Configure the runtime defaults applied by app-standards. Publish with `php artisan vendor:publish --tag=mortelos-app-standards-config`. |

## Boundaries

`mortelos/app-standards` owns app runtime defaults and the Spine contract text and config. It does not own portal features, domain policy, tenant branding, connectors, widgets, dashboards or MCP tools. The checks that enforce the Spine live in `mortelos/dev-tools`. The package does not depend on `mortelos/framework`: the Spine config names framework classes as strings.

Reusable packages should remain installable without these defaults. Host apps opt into the standards package directly.
