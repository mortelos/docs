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
last_verified: "2026-06-04"
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

## Verify

```bash
php artisan chat:widget:check compliance_intake_start
```

## Boundaries

Use this package for compliance-specific widget UI and registration. Keep generic widget runtime logic in `mortelos/chat`, and keep irreversible compliance writes behind policy checks and approval flows.
