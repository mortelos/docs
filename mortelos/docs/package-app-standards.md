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
last_verified: "2026-08-27"
public: true
---

# MortelOS App Standards

`mortelos/app-standards` owns the Laravel runtime defaults every MortelOS host app should start with.

This package is intentionally a MortelOS-owned hard-fork-style reimplementation, not a runtime dependency on `nunomaduro/essentials`. Upstream packages and community conventions can inspire defaults, but MortelOS owns the behavior, names, tests and release cadence.

## Why It Exists

App-wide Laravel defaults are powerful. They affect Eloquent, validation, URLs, console safety and test behavior. Putting those defaults in a small MortelOS package gives every host app the same baseline without making reusable feature packages carry app-level side effects.

Use this package in host apps such as `mortelos/starter`, `uteq/os`, `uteq/sijperda-os` and other concrete OS installations. Do not require it from reusable packages such as `mortelos/framework`, channels, widgets or studio packages.

## Defaults

| Area | Default |
| --- | --- |
| Eloquent | Strict models are enabled. |
| Relationships | Automatic relationship eager loading is enabled. |
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

To test a local checkout of the package against a host app, symlink it into `vendor/` after installing. Do not add a `path` repository to the committed `composer.json`; see [Packages](/docs/0/packages).

## Configuration

Publish the config when a host needs to override a default:

```bash
php artisan vendor:publish --tag=mortelos-app-standards-config
```

The published file is `config/app-standards.php`.

Prefer environment-specific toggles over removing the package. If a default is wrong for most MortelOS hosts, change it in this package and cover the behavior with a package test.

## Boundaries

`mortelos/app-standards` owns app runtime defaults. It does not own portal features, domain policy, tenant branding, connectors, widgets, dashboards or MCP tools.

Reusable packages should remain installable without these defaults. Host apps opt into the standards package directly.
