---
title: "MortelOS Triage"
nav_title: "Triage"
slug: "package-triage"
version: "0"
description: "Building blocks for a mail triage loop: one judgement per thread, a doubt check and a receipt per decision."
section: "packages"
order: 62
status: "mvp"
audience: "developers"
package: "mortelos/triage"
canonical_path: "/docs/0/package-triage"
last_verified: "2026-10-14"
public: true
---

# MortelOS Triage

`mortelos/triage` provides the building blocks of a triage loop for incoming mail: one judgement per thread, a doubt check and a receipt for every decision.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Judgement | The `Oordeelaar` contract: one judgement per mail thread, with a route, a confidence, whether the dossier was enough, a doubt and advice. The host implements it. |
| Doubt check | `Uitsmijter` says when the loop should not act but ask one question: below the confidence threshold, or when the dossier was not enough. |
| Threshold | `Instellingen::drempel()` reads the threshold from the tenant settings, falls back to config and keeps it between 0 and 1. |
| Receipts | `Bonnetje` records each decision and its outcome as an `ai_decision` entity through the framework decision actions. |
| Test cards | `triage:proef` runs a folder of test cards through the bound `Oordeelaar` and compares the result with what a person chose. |

The names in the code are Dutch: `Oordeelaar` (judge), `Uitsmijter` (bouncer, the doubt check), `Bonnetje` (receipt) and `drempel` (threshold). What differs per installation is data, not code: rules, labels, prompt text, threshold and addresses.

## Install

```bash
composer require mortelos/triage
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The package needs `mortelos/framework` 0.6.36 or later. The service provider is auto-discovered.

## Commands

```bash
php artisan triage:proef --map=<folder> --drempel=0.6 --vandaag=2026-10-14
```

Every option is optional. Without `--map` the command reads the `proef/` folder of the package; point `--map` at your own cards. Each card is a JSON file with `naam`, `thread` and `verwacht.route`. The command fails when no `Oordeelaar` is bound. It knows nothing about tenants, so the host runs it inside a tenant.

## Outlets

An outlet is a place where you extend `mortelos/triage` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.1.2; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list triage
```

| Kind | Name | What it is for |
| --- | --- | --- |
| contract | `Mortelos\Triage\Contracts\Oordeelaar` | Gives one judgement on one mail thread. No default: the host must bind it. |
| config | `triage` | The confidence threshold (`drempel`) below which the loop asks one question instead of acting. Override it per tenant in `tenant_settings` with the key `triage.drempel`. Publish with `php artisan vendor:publish --tag=triage-config`. |

## Boundaries

The package owns the judgement contract, the doubt check, the threshold and the receipts. The host app owns the `Oordeelaar` (which model, which prompt, which tools it may read with), the mail source, the loop that ties the pieces together, tenancy and the actions a decision leads to.
