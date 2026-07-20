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
last_verified: "2026-07-20"
public: true
---

# Starter Package

`mortelos/starter` is the Laravel host app template for a governed MortelOS portal.

It gives you a working application first. You get login, dashboard, inbox, governance, users, settings and diagnostics before you add customer-specific capability code.

## Included baseline

| Area | Included |
| --- | --- |
| Framework | Laravel 13, Livewire 4, Flux UI, Tailwind and Pest |
| Runtime standards | `mortelos/app-standards` for host-level Laravel defaults |
| Auth baseline | Password login, invitation stub and passkey stub |
| Shell routes | `/login`, `/dashboard`, `/inbox`, `/governance`, `/users`, `/settings`, `/onboarding` |
| Layout | `mortelos-starter::layouts.app` and `layouts.guest` |
| Livewire namespace | `starter::` pages and shared shell components |
| Config contracts | `config/starter.php` with boot-safe defaults |
| Event store | `events` table and MortelOS event-sourcing config |
| Tenant baseline | Default tenant plus explicit `tenant_user` membership for the seed account |
| Seed account | `admin@example.test` / `password` |
| Diagnostics | `php artisan starter:doctor` |
| Tests | Pest boot smoke and config shape tests |
| Agent guidance | `AGENTS.md`, `mortelos/agent-standards`, `docs/building-portals.md`, `knowledge/` and the `setup-portal` skill |

## Extension model

Portal-specific behavior is added through host bindings, resolvers, actions, policies, projections and package decisions.

Use packages for behavior that can serve more than one MortelOS installation. Keep tenant policy, branding and local orchestration in the host.

Host-wide framework defaults live in `mortelos/app-standards`, not in reusable feature packages. Shared agent instructions live in `mortelos/agent-standards` and are merged into `AGENTS.md` through `mortelos/dev-tools`.

## Required auth contracts

Required auth contracts already point at working stubs, so a fresh app boots. Replace the stubs as the portal's auth flow becomes specific.

| Config key | Default responsibility |
| --- | --- |
| `auth.post_login_redirect_resolver` | Returns the post-login URL, `/dashboard` by default. |
| `auth.controllers.password_login` | Handles email and password login. |
| `auth.controllers.passkey_authenticated` | Stub for passkey login POST. |
| `auth.controllers.accept_invitation` | Stub for invitation show and store. |
| `users.resolver` | Lists members of the configured tenant and handles invitation placeholders. |
| `users.access_resolver` | Guards user inspection actions. |

Optional resolvers for sidebar navigation, universal search, governance, onboarding, inbox item types and dashboard messages can stay `null` until the capability map calls for them.

The `users` table stores identities. Membership and the active tenant role live
in `tenant_user`; a user without a row for the configured tenant is not a member
and must not inherit access from `role_user` or a default role name.

## Verification

Every host change should keep the boot baseline intact:

```bash
php artisan starter:doctor
vendor/bin/pest
```

The manual smoke is login to dashboard with `admin@example.test` / `password`.
