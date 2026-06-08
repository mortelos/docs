---
title: "MortelOS Chat"
nav_title: "Chat"
slug: "package-chat"
version: "0"
description: "Tenant-gated chat workspace and widget runtime for MortelOS."
section: "packages"
order: 54
status: "mvp"
audience: "developers"
package: "mortelos/chat"
canonical_path: "/docs/0/package-chat"
last_verified: "2026-06-08"
public: true
---

# MortelOS Chat

`mortelos/chat` provides the tenant-gated chat workspace and the chat widget runtime.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Chat workspace | Routes, views and Livewire components for a tenant-gated workspace. |
| Widget runtime | `WidgetRegistry`, `WidgetRenderer`, widget definitions and widget run persistence. |
| Connector setup widgets | Render provider-driven connector setup forms and OAuth next actions from channel packages. |
| Widget commands | `chat:widget:make` and `chat:widget:check` for scaffolding and validation. |
| Shared storage | The `chat_widget_runs` tenant migration for widget runtime state. |

## Install

```bash
composer require mortelos/chat
```

## Commands

```bash
php artisan chat:widget:make demo_widget
php artisan chat:widget:check compliance_intake_start
```

## Boundaries

Use this package for generic chat infrastructure. Put domain-specific widgets in dedicated packages such as `mortelos/widget-compliance` or `mortelos/widget-document-feedback`.
