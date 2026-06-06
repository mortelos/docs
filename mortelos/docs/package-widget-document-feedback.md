---
title: "Document Feedback Widget"
nav_title: "Document feedback"
slug: "package-widget-document-feedback"
version: "0"
description: "Document feedback widget for MortelOS chat surfaces."
section: "packages"
order: 64
status: "mvp"
audience: "developers"
package: "mortelos/widget-document-feedback"
canonical_path: "/docs/0/package-widget-document-feedback"
last_verified: "2026-06-04"
public: true
---

# Document Feedback Widget

`mortelos/widget-document-feedback` provides document feedback flows inside the MortelOS chat surface.

## Registered Widget

| Field | Value |
| --- | --- |
| Key | `document_feedback_annotate` |
| Skill | `document_feedback` |
| Livewire component | `widget-document-feedback::annotator` |
| Allowed actions | `add_annotation`, `ignore_annotation`, `prepare_proposal`, `apply_proposal` |

## Install

```bash
composer require mortelos/widget-document-feedback
```

## Verify

```bash
php artisan chat:widget:check document_feedback_annotate
```

## Boundaries

Use this package for document feedback widget registration and UI. Keep generic widget runtime logic in `mortelos/chat`, and keep local document storage, approval policy and apply behavior in the host workflow unless those parts become reusable.

