---
title: "Starter Package"
nav_title: "Starter package"
slug: "starter-package"
version: "0"
description: "What the mortelos/starter host app template provides."
section: "reference"
order: 60
status: "mvp"
audience: "developers"
package: "mortelos/starter"
canonical_path: "/docs/0/starter-package"
last_verified: "2026-05-31"
public: true
---

# Starter Package

`mortelos/starter` is the Laravel host app template for a governed MortelOS portal.

It gives you a working application first. You get login, tenant selection, dashboard, inbox, governance, users, settings and diagnostics before you add customer-specific capability code.

## Included baseline

| Area | Included |
| --- | --- |
| Framework | Laravel, Livewire, Flux UI, Tailwind and Pest |
| Auth baseline | Password login, invitation stub, passkey stub and tenant-select stub |
| Shell routes | `/login`, `/dashboard`, `/inbox`, `/governance`, `/users`, `/settings`, `/onboarding` |
| Layout | App and guest layouts for portal surfaces |
| Livewire namespace | Starter pages and shared shell components |
| Config contracts | `config/starter.php` with boot-safe defaults |
| Diagnostics | `php artisan starter:doctor` |
| Agent guidance | `AGENTS.md`, portal build docs and kickoff guidance |

## Extension model

Portal-specific behavior is added through host bindings, resolvers, actions, policies, projections and package decisions.

Use packages for behavior that can serve more than one MortelOS installation. Keep tenant policy, branding and local orchestration in the host.

