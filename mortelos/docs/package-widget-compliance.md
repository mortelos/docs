---
title: "Compliance Widget"
nav_title: "Compliance widget"
slug: "package-widget-compliance"
version: "0"
description: "Compliance-focused chat widgets for MortelOS."
section: "widgets"
order: 63
status: "mvp"
audience: "developers"
package: "mortelos/widget-compliance"
canonical_path: "/docs/0/package-widget-compliance"
last_verified: "2026-10-16"
public: true
---

# Compliance Widget

`mortelos/widget-compliance` provides compliance-focused chat widgets for MortelOS.

## Registered Widget

| Field | Value |
| --- | --- |
| Key | `compliance_intake_start` |
| Skill | `compliance_intake` |
| Livewire component | `widget-compliance::compliance-intake` |
| Policy ability | `compliance.intake.start` |

## Install

```bash
composer require mortelos/widget-compliance
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Verify

```bash
php artisan chat:widget:check compliance_intake_start
```

## Upgrading to 0.4.0

Require `mortelos/framework:^0.7` and `mortelos/chat:^0.6`, run the framework upgrade migration, and update chat and host integrations together. See [Upgrading to 0.7.0](/docs/0/package-framework#upgrading-to-070) for the migration.

Update host-provided Livewire components and published intake views. There are no aliases for the old names.

| 0.3 | 0.4 |
| --- | --- |
| `compliance.normkaart-publication-widget` | `compliance.rulebook-publication-widget` |
| `compliance.bronnen-toevoegen-widget` | `compliance.add-sources-widget` |
| compliance visibility key `normkaart` | `rulebook` |
| view outlet `widget-compliance.intake.normkaart-publication` | `widget-compliance.intake.rulebook-publication` (update registrations that use `Outlet`) |
| framework model `Mortel\Models\ComplianceNormkaartVersion` | `Mortel\Models\ComplianceRulebookVersion` |

Dutch customer-facing copy stays Dutch.

## Outlets

An outlet is a place where you extend `mortelos/widget-compliance` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.4.0; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list widget-compliance
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| view | `widget-compliance.intake.start` | The start card of the compliance intake, while there is no sources panel or review package yet. Default: the Livewire component `compliance.intake-start-widget`, which the host provides. | `runData`, `steps` |
| view | `widget-compliance.intake.sources` | The panel for adding sources to the intake. Default: `compliance.add-sources-widget`, which the host provides. | `runId`, `compact` |
| view | `widget-compliance.intake.review-package` | The review package of the intake. Default: `compliance.review-package-widget`, which the host provides. | `runId` |
| view | `widget-compliance.intake.workflow-review` | The review of the workflow links the intake proposes. Default: `compliance.workflow-review-widget`, which the host provides. | `runId` |
| view | `widget-compliance.intake.open-questions` | The open questions of the intake. Default: `compliance.open-questions-widget`, which the host provides. | `runId` |
| view | `widget-compliance.intake.rulebook-publication` | The publication of the rulebook. Default: `compliance.rulebook-publication-widget`, which the host provides. | `runId` |

## Boundaries

Use this package for compliance-specific widget UI and registration. Keep generic widget runtime logic in `mortelos/chat`, and keep irreversible compliance writes behind policy checks and approval flows.
