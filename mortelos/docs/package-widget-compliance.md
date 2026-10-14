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
last_verified: "2026-10-14"
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

## Outlets

An outlet is a place where you extend `mortelos/widget-compliance` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.3.2; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list widget-compliance
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| view | `widget-compliance.intake.start` | The start card of the compliance intake, while there is no sources panel or review package yet. Default: the Livewire component `compliance.intake-start-widget`, which the host provides. | `runData`, `steps` |
| view | `widget-compliance.intake.sources` | The panel for adding sources to the intake. Default: `compliance.bronnen-toevoegen-widget`, which the host provides. | `runId`, `compact` |
| view | `widget-compliance.intake.review-package` | The review package of the intake. Default: `compliance.review-package-widget`, which the host provides. | `runId` |
| view | `widget-compliance.intake.workflow-review` | The review of the workflow links the intake proposes. Default: `compliance.workflow-review-widget`, which the host provides. | `runId` |
| view | `widget-compliance.intake.open-questions` | The open questions of the intake. Default: `compliance.open-questions-widget`, which the host provides. | `runId` |
| view | `widget-compliance.intake.normkaart-publication` | The publication of the norm card. Default: `compliance.normkaart-publication-widget`, which the host provides. | `runId` |

## Boundaries

Use this package for compliance-specific widget UI and registration. Keep generic widget runtime logic in `mortelos/chat`, and keep irreversible compliance writes behind policy checks and approval flows.
