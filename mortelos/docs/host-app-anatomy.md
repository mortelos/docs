---
title: "Host App Anatomy"
nav_title: "Host app anatomy"
slug: "host-app-anatomy"
version: "0"
description: "Where MortelOS host app behavior belongs on disk."
section: "build-method"
order: 32
status: "mvp"
audience: "developers"
package: "mortelos/starter"
canonical_path: "/docs/0/host-app-anatomy"
last_verified: "2026-06-07"
public: true
---

# Host App Anatomy

`mortelos/starter` starts as a runnable Laravel host. Portal work grows it by adding host-owned actions, resolvers, policies, projections and package decisions.

## Boot baseline

| Path | Ships | Purpose |
| --- | --- | --- |
| `app/Actions/Auth/ResolvePostLoginRedirect.php` | Yes | Returns `/dashboard` by default. |
| `app/Console/Commands/StarterDoctor.php` | Yes | Checks boot baseline wiring. |
| `app/Http/Controllers/Auth/PasswordLoginController.php` | Yes | Working email and password login. |
| `app/Http/Controllers/Auth/PasskeyAuthenticatedController.php` | Yes | Replace when passkeys are specified. |
| `app/Http/Controllers/Auth/AcceptInvitationController.php` | Yes | Replace when invitation persistence is specified. |
| `app/Support/StarterUsersResolver.php` | Yes | Lists local users and exposes invitation placeholders. |
| `app/Support/StarterUsersAccessResolver.php` | Yes | Guards user inspection actions. |
| `config/starter.php` | Yes | Host contract surface with safe defaults. |
| `routes/starter.php` | Yes | Starter route bridge. |
| `routes/web.php` | Yes | Requires `routes/starter.php`. |
| `tests/Feature/BootSmokeTest.php` | Yes | Confirms login and dashboard baseline. |
| `tests/Feature/ConfigShapeTest.php` | Yes | Confirms required config shape. |

## What lives where

| Concern | Lives in | Notes |
| --- | --- | --- |
| Layout shell | `resources/views/layouts/app.blade.php` and `guest.blade.php` | Keep the starter layout instead of replacing it. |
| Auth pages | `resources/views/livewire/pages/auth/` | Livewire 4 single-file components. |
| Auth logic | `app/Http/Controllers/Auth/` | Replace stubs when the portal auth flow is specified. |
| Post-login redirect | `app/Actions/Auth/ResolvePostLoginRedirect.php` | Make role-aware per portal when needed. |
| Tenant resolution | Host-owned support class | The host picks its tenancy strategy. |
| Navigation tree | `app/Support/StarterSidebarNavigationResolver.php` | Bind through `navigation.sidebar_resolver`. |
| Universal search | `app/Support/StarterUniversalSearchResolver.php` | Bind through `navigation.universal_search_resolver`. |
| Governance access | `app/Support/StarterGovernanceAccessResolver.php` | Bind through `governance.access_resolver`. |
| User management | `app/Support/StarterUsersResolver.php` | Bind through `users.resolver`. |
| Dashboard widgets | Host Livewire component or package | Decide with package governance. |
| Policy abilities | `app/Policies/` | Deny by default. |
| Portal docs | `docs/portals/<slug>/` | Capability map, build plan and progress. |
| Package decisions | `.mortelos/package-decisions.md` | One entry per surface or package boundary. |
| MCP mount | `routes/ai.php` | Host mounts the framework MCP server when operate mode is enabled. |

## Expected portal slice output

A complete first vertical slice usually adds:

1. `docs/portals/<slug>/capability-map.md`.
2. A package decision in `.mortelos/package-decisions.md`.
3. Actions for write paths.
4. Events for auditable facts.
5. A projection or read model for the surface.
6. A surface such as dashboard widget, package route, page widget, chat widget or inbox item.
7. Deny-by-default policy abilities.
8. Focused Pest feature tests.
9. Verification evidence from doctor, Pest and manual smoke.

## Test account

The template seeds one local account:

| Role | Email | Password |
| --- | --- | --- |
| Admin | `admin@example.test` | `password` |

Add account-manager, customer or reviewer accounts in the host seeder only after the portal's role model is specified.
