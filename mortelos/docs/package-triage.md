---
title: "MortelOS Triage"
nav_title: "Triage"
slug: "package-triage"
version: "0"
description: "Building blocks for a mail triage loop: one assessment per thread, a doubt check and a decision log entry per decision."
section: "packages"
order: 62
status: "mvp"
audience: "developers"
package: "mortelos/triage"
canonical_path: "/docs/0/package-triage"
last_verified: "2026-10-16"
public: true
---

# MortelOS Triage

`mortelos/triage` provides the building blocks of a triage loop for incoming mail: one assessment per thread, a doubt check and a decision log entry for every decision.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Judgement | The `Assessor` contract: one assessment per mail thread, with a route, a confidence, whether the context was sufficient, a doubt and advice. The host implements it. |
| Doubt check | `ConfidenceGate` says when the loop should not act but ask one question: below the confidence threshold, or when the context was not sufficient. |
| Threshold | `Settings::threshold()` reads the threshold from the tenant settings, falls back to config and keeps it between 0 and 1. |
| Decision log | `DecisionLog` records each decision and its outcome as an `ai_decision` entity through the framework decision actions. |
| Evaluation | `triage:eval` runs a folder of eval cases through the bound `Assessor` and compares the result with what a person chose. |

What differs per installation is data, not code: rules, labels, prompt text, threshold and addresses.

## Install

```bash
composer require mortelos/triage
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The package needs `mortelos/framework` 0.6.36 or later, in the 0.6 series or the 0.7 series. The service provider is auto-discovered.

## Commands

```bash
php artisan triage:eval --directory=<folder> --threshold=0.6 --today=2026-10-14
```

Every option is optional. Without `--directory` the command reads the `eval/` folder of the package; point `--directory` at your own cases. Each case is a JSON file with `name`, `thread` and `expected.route`. The command fails when no `Assessor` is bound. It knows nothing about tenants, so the host runs it inside a tenant.

## Upgrading to 0.2.0

Version 0.2.0 is a minor bump with breaking public renames. There are no compatibility aliases. Update imports, contract bindings, calls, named arguments, published config, tenant settings, assessor output, eval cases and code that reads evaluation reports. The framework constraint stays `^0.6.36`; version 0.2.1 also allows framework 0.7.

### Classes, methods and constants

All names are relative to `Mortelos\Triage\`. Source files and test files are renamed with their classes.

| Old | New |
| --- | --- |
| `Uitsmijter` | `ConfidenceGate` |
| `Uitsmijter::onzeker()` | `ConfidenceGate::isUncertain()` |
| `Uitsmijter::vraag()` | `ConfidenceGate::question()` |
| `Uitsmijter::STANDAARD_VRAAG` | `ConfidenceGate::DEFAULT_QUESTION` |
| `Bonnetje` | `DecisionLog` |
| `Bonnetje::schrijf()` | `DecisionLog::record()` |
| `Bonnetje::uitkomst()` | `DecisionLog::outcome()` |
| `Bonnetje::lees()` | `DecisionLog::read()` |
| `Bonnetje::HANDELINGEN` | `DecisionLog::ACTIONS` |
| `Bonnetje::UITKOMSTEN` | `DecisionLog::OUTCOMES` (English values only, now a list) |
| `Contracts\Oordeelaar` | `Contracts\Assessor` |
| `Contracts\Oordeelaar::oordeel()` | `Contracts\Assessor::assess()` |
| `Instellingen` | `Settings` |
| `Instellingen::drempel()` | `Settings::threshold()` |
| `Proef\Proef` | `Evaluation\Evaluation` |
| `Proef\Proef::draai()` | `Evaluation\Evaluation::run()` |
| `Console\ProefCommand` | `Console\EvaluationCommand` |

Named arguments also change: `$oordeel` to `$assessment`, `$drempel` to `$threshold`, `$beslissing` to `$decision`, `$eigenaarId` to `$ownerId`, `$bonnetjeId` to `$decisionId`, `$uitkomst` to `$outcome`, `$optie` to `$option`, `$noot` to `$note`, `$oordeelaar` to `$assessor`, `$instellingen` to `$settings`, `$map` to `$directory`, `$vandaag` to `$today`, `$projecten` to `$projects`, and the constructor argument `$uitkomstActie` to `$recordOutcome`. Other public parameter names are unchanged.

### Config and command

| Old | New |
| --- | --- |
| config key `drempel` in `config/triage.php` | `confidence_threshold` |
| config and `tenant_settings` key `triage.drempel` | `triage.confidence_threshold` |
| environment variable `TRIAGE_DREMPEL` | `TRIAGE_CONFIDENCE_THRESHOLD` |
| `triage:proef` | `triage:eval` |
| option `--map` | `--directory` |
| option `--drempel` | `--threshold` |
| option `--vandaag` | `--today` |
| bundled case directory `proef/` | `eval/` |
| saved report directory `triage-proef/` on the local disk | `triage-eval/` |

Rename existing tenant setting keys. Update published config and environment variables before you clear or rebuild the config cache. The default threshold stays `0.6`. Move custom case directories, and update automation that runs the command or reads saved reports.

### Payload and report keys

Apply these renames to the input of `DecisionLog::record()` and the results of `Assessor::assess()`. `ConfidenceGate` reads the same renamed assessment keys.

| Old | New |
| --- | --- |
| `soort` | `kind` |
| `onderwerp_type` | `subject_type` |
| `onderwerp_id` | `subject_id` |
| `verwijzing` | `reference` |
| `afzender` | `sender` |
| `samenvatting` (decision input) | `input_summary` |
| `bronnen` | `sources` |
| `regels` (decision input) | `rules` |
| `oordeel` | `assessment` |
| `handeling` | `action` |
| `zekerheid` | `confidence` |
| `dossier_voldoende` | `context_sufficient` |
| `twijfel` | `doubt` |
| `reden` | `reason` |
| `keuze` | `choice` |
| `opties` | `options` |
| `advies` | `advice` |
| `concept` | `draft` |

As before, `draft`, `agent_prompt` and `sources` are left out of the stored assessment output. The framework storage schema of `ai_decision` and its top-level fields, including `sufficient`, are unchanged. Update consumers of nested assessment output keys. The package does not rewrite historical records.

Evaluation JSON and returned reports use these extra renames:

| Old | New |
| --- | --- |
| `naam` | `name` |
| `projecten` | `projects` |
| `verwacht` | `expected` |
| `verwacht.keuze` | `expected.choice` |
| `genoeg_info` | `enough_info` |
| `opmerking` | `note` |
| `eerder` | `previous` |
| `titel` | `title` |
| `keuze_advies` | `choice_advice` |
| `regels` (report) | `rows` |
| `samenvatting` (report) | `summary` |
| `verwacht_route` | `expected_route` |
| `route_gelijk` | `route_matches` |
| `onzeker` | `uncertain` |
| `keuze_mens` | `human_choice` |
| `aantal` | `count` |
| `routes_gelijk` | `matching_routes` |
| `gemiddelde_zekerheid` | `average_confidence` |

The report also uses `confidence`, `context_sufficient`, `doubt` and `advice` from the table above. Captured message content, route labels and human choices stay as they were.

## Outlets

An outlet is a place where you extend `mortelos/triage` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.2.1; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list triage
```

| Kind | Name | What it is for |
| --- | --- | --- |
| contract | `Mortelos\Triage\Contracts\Assessor` | Assesses one mail thread. No default: the host must bind it. |
| config | `triage` | The confidence threshold (`confidence_threshold`) below which the loop asks one question instead of acting. Override it per tenant in `tenant_settings` with the key `triage.confidence_threshold`. Publish with `php artisan vendor:publish --tag=triage-config`. |

## Boundaries

The package owns the assessment contract, the doubt check, the threshold and the decision log. The host app owns the `Assessor` (which model, which prompt, which tools it may read with), the mail source, the loop that ties the pieces together, tenancy and the actions a decision leads to.
