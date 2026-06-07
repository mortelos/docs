---
title: "Starter Package"
nav_title: "Starter package"
slug: "starter-package"
version: "0"
description: "What the mortelos/starter host app template provides."
section: "packages"
order: 51
status: "mvp"
audience: "developers"
package: "mortelos/starter"
canonical_path: "/docs/0/starter-package"
last_verified: "2026-06-07"
public: true
---

# Starter Package

`mortelos/starter` is the Laravel host app template for a governed MortelOS portal.

It gives you a working application first. You get login, dashboard, inbox, governance, users, settings and diagnostics before you add customer-specific capability code.

## Included baseline

| Area | Included |
| --- | --- |
| Framework | Laravel 13, Livewire 4, Flux UI, Tailwind and Pest |
| Auth baseline | Password login, invitation stub and passkey stub |
| Shell routes | `/login`, `/dashboard`, `/inbox`, `/governance`, `/users`, `/settings`, `/onboarding` |
| Layout | `mortelos-starter::layouts.app` and `layouts.guest` |
| Livewire namespace | `starter::` pages and shared shell components |
| Config contracts | `config/starter.php` with boot-safe defaults |
| Event store | `events` table and MortelOS event-sourcing config |
| Seed account | `admin@example.test` / `password` |
| Diagnostics | `php artisan starter:doctor` |
| Tests | Pest boot smoke and config shape tests |
| Agent guidance | `AGENTS.md`, `docs/building-portals.md`, `knowledge/` and the `setup-portal` skill |

## Extension model

Portal-specific behavior is added through host bindings, resolvers, actions, policies, projections and package decisions.

Use packages for behavior that can serve more than one MortelOS installation. Keep tenant policy, branding and local orchestration in the host.

## Required auth contracts

Required auth contracts already point at working stubs, so a fresh app boots. Replace the stubs as the portal's auth flow becomes specific.

| Config key | Default responsibility |
| --- | --- |
| `auth.post_login_redirect_resolver` | Returns the post-login URL, `/dashboard` by default. |
| `auth.controllers.password_login` | Handles email and password login. |
| `auth.controllers.passkey_authenticated` | Stub for passkey login POST. |
| `auth.controllers.accept_invitation` | Stub for invitation show and store. |
| `users.resolver` | Lists local users and handles invitation placeholders. |
| `users.access_resolver` | Guards user inspection actions. |

Optional resolvers for sidebar navigation, universal search, governance, onboarding, inbox item types and dashboard messages can stay `null` until the capability map calls for them.

## Verification

Every host change should keep the boot baseline intact:

```bash
php artisan starter:doctor
vendor/bin/pest
```

The manual smoke is login to dashboard with `admin@example.test` / `password`.
