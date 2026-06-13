---
title: "TALL Conventions"
nav_title: "TALL conventions"
slug: "tall-conventions"
version: "0"
description: "Frontend and Laravel implementation conventions for MortelOS host apps."
section: "build-method"
order: 33
status: "mvp"
audience: "developers"
package: "mortelos/starter"
canonical_path: "/docs/0/tall-conventions"
last_verified: "2026-06-13"
public: true
---

# TALL Conventions

MortelOS Starter uses Laravel 13, Livewire 4 single-file components, Flux UI, Tailwind and Pest.

## Rules

| Rule | Why |
| --- | --- |
| Check Flux UI first before writing custom Alpine or Tailwind. | Flux is the design system. |
| Use Livewire 4 single-file components for new portal surfaces. | State, lifecycle and view stay together. |
| Put write paths in action classes under `app/Actions/<Domain>/`. | Domain rules stay out of components. |
| Use Pest for tests. | The starter baseline and capability tests run through Pest. |
| Use Pint for formatting. | One style across host and packages. |
| Deny by default for policies. | Governance remains consistent across web, widgets and agents. |

## Livewire surface shape

New host-owned pages and widgets usually live under:

```text
resources/views/livewire/
  pages/<domain>/<name>.blade.php
  shared/<component>.blade.php
```

Package-owned surfaces use the package's Livewire namespace and should not hardcode host-specific classes. Use config and resolver contracts for host-specific behavior.

## Domain boundaries

Do not embed domain rules in Blade or Livewire components. Components should:

1. Read projection or service state.
2. Render Flux-aligned UI.
3. Call action classes for writes.
4. Let policies and resolvers decide access.

Actions, projections, policies, resolvers and package services own behavior.

## Surface selection

Use the smallest useful surface:

| Surface | Use for |
| --- | --- |
| Dashboard widget | Dense operational overview or queue status. |
| Inbox item | Human review, approval or decision point. |
| Page widget | Reusable block embedded in a host page. |
| Chat widget | Guided task inside `mortelos/chat`. |
| Package route | Reusable feature workspace. |

Do not add a standalone page when a dashboard widget or inbox item fits the workflow.

## Frontend technology choice

Livewire 4 single-file components are the default for every portal surface. Blaze and islands cover most cases that previously felt too interactive for Livewire: Blaze speeds up Blade rendering on the server, and islands scope a re-render to a single region so only that part takes a roundtrip. Alpine handles client-only interactions (toggles, conditional fields, small calculations) without any roundtrip.

These tools raise the bar for reaching outside Livewire, but they do not change where state lives. Blaze and islands still hit the server; Alpine is a light reactivity layer, not a full client framework. Reach for Vue only when the state is inherently client-side and Alpine is too thin for it:

| Signal | Why Livewire and Alpine are the wrong fit |
| --- | --- |
| Offline or optimistic UI | State must live on the client; a server roundtrip is unavailable or must not block the UI. |
| High-frequency or collaborative updates (live cursors, realtime canvas) | A roundtrip per frame is too slow even when islands keep each one light. |
| Heavy client interaction (graph editor with pan, zoom, drag; complex diagram editing) | Alpine is too thin for large client state; this is component-framework territory. |

If none of these apply, stay on Livewire 4. Introducing Vue is itself an architecture decision: record a `mortelos:package-decision` before adding a Vue surface, and keep it as an island mounted inside the Livewire shell rather than a separate SPA.

## Verification

For UI work, verify:

```bash
vendor/bin/pint --dirty
php artisan starter:doctor
vendor/bin/pest
```

Then open the affected surface manually and check responsive layout, allowed access and denied access.
