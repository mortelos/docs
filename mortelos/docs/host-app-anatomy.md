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
last_verified: "2026-10-14"
public: true
---

# Host App Anatomy

`mortelos/starter` starts as a runnable Laravel host. Portal work grows it by adding host-owned actions, aggregates, events, projectors, resolvers, policies and package decisions.

## Boot baseline

| Path | Ships | Purpose |
| --- | --- | --- |
| `app/Actions/Auth/ResolvePostLoginRedirect.php` | Yes | Returns `/dashboard` by default. |
| `app/Actions/Invitations/` | Yes | `InviteUser`, `RevokeInvitation` and `AcceptInvitation`: the invitation writes, each returning an `ActionResult`. |
| `app/Aggregates/InvitationAggregate.php` | Yes | Holds the invitation guards and records the events. |
| `app/Events/Invitations/` | Yes | `InvitationCreated`, `InvitationRevoked` and `InvitationAccepted`, each with a `*.v1.schema.json` next to it. |
| `app/Projectors/InvitationProjector.php` | Yes | The only writer of the `invitations` table. |
| `app/Models/Invitation.php` | Yes | Read model of an invitation; change it through the Actions, never on the model. |
| `app/Access/StarterGovernanceGate.php` | Yes | Deny-by-default gate over role and policy data; `SCOPES` lists the policy scope of each host ability. |
| `app/Console/Commands/StarterDoctor.php` | Yes | Checks boot baseline wiring. |
| `app/Http/Controllers/Auth/PasswordLoginController.php` | Yes | Working email and password login. |
| `app/Http/Controllers/Auth/PasskeyAuthenticatedController.php` | Yes | Replace when passkeys are specified. |
| `app/Http/Controllers/Auth/AcceptInvitationController.php` | Yes | Serves `/invite/{token}`: shows the form, then calls `AcceptInvitation` and logs the new user in. |
| `app/Support/StarterUsersResolver.php` | Yes | Lists members of the configured tenant and the open invitations. |
| `app/Support/StarterUsersAccessResolver.php` | Yes | Guards user inspection actions. |
| `app/Support/StarterTenantInvitationService.php` | Yes | Implements the framework `TenantInvitationService` contract by calling `InviteUser`. |
| `config/starter.php` | Yes | Host contract surface with safe defaults. |
| `config/mortel.php` | Yes | Host overrides for the framework; registers `App\Events\` as an event schema namespace. |
| `config/spine.php` | Yes | Governed models and the classes allowed to write them. |
| `config/event-sourcing.php` | Yes | `event_class_map` for host events and the framework projector list. |
| `config/app-standards.php` | Optional | Published overrides for `mortelos/app-standards`. |
| `routes/starter.php` | Yes | Starter route bridge, with `Route::livewire('/users', 'pages::users.index')` style routes. |
| `routes/web.php` | Yes | Requires `routes/starter.php`. |
| `tests/Feature/BootSmokeTest.php` | Yes | Confirms login and dashboard baseline. |
| `tests/Feature/ConfigShapeTest.php` | Yes | Confirms required config shape. |
| `tests/Feature/ArchGuardTest.php` | Yes | Fails on writes outside Actions, events outside aggregates and Livewire outside the Livewire 4 paths. |
| `tests/Feature/InvitationsTest.php` | Yes | Covers the invitation Actions, the undo and the projection replay. |

## What lives where

| Concern | Lives in | Notes |
| --- | --- | --- |
| Layout shell | `resources/views/layouts/app.blade.php` and `guest.blade.php` | Used as `layouts::app` and `layouts::guest`. Keep the starter layout instead of replacing it. |
| Pages | `resources/views/pages/<domain>/⚡<name>.blade.php` | Routed as `pages::<domain>.<name>`. |
| Auth pages | `resources/views/pages/auth/⚡login.blade.php` | Livewire 4 single-file component, routed as `pages::auth.login`. |
| Auth logic | `app/Http/Controllers/Auth/` | Password login and invitation acceptance work; replace the passkey stub when it is specified. |
| Post-login redirect | `app/Actions/Auth/ResolvePostLoginRedirect.php` | Make role-aware per portal when needed. |
| Tenant resolution | `app/Support/SingleTenantResolver.php` | The starter resolves one tenant; the host picks its own tenancy strategy and rebinds `TenantResolver`. |
| Navigation tree | Host-owned resolver class | Bind through `navigation.sidebar_resolver` in `config/starter.php`; `null` by default. |
| Universal search | Host-owned resolver class | Bind through `navigation.universal_search_resolver` in `config/starter.php`; `null` by default. |
| Governance access | `app/Access/StarterGovernanceGate.php` | Bound to the `GovernanceGate` contract in `app/Providers/StarterServiceProvider.php`. |
| User management | `app/Support/StarterUsersResolver.php` | Bind through `users.resolver`; scope reads and inspection to explicit `tenant_user` membership. |
| Dashboard widgets | `resources/views/components/<group>/⚡<name>.blade.php` or a package | Decide with package governance. |
| Policy abilities | `StarterGovernanceGate::SCOPES` in `app/Access/StarterGovernanceGate.php` | Host abilities use `PolicyScope::Action`. Policies are framework rows created through `CreatePolicy`; deny by default. |
| Host facts | `app/Aggregates/`, `app/Events/<Domain>/`, `app/Projectors/` | A fact the host must audit or undo: aggregate, stored event, projector. See [TALL conventions](/docs/0/tall-conventions). |
| Portal docs | `docs/portals/<slug>/` | Capability map, build plan and progress. |
| Package decisions | `.mortelos/package-decisions.md` | One entry per surface or package boundary. |
| Agent rules | `AGENTS.md` | Host-specific prose plus generated package rules from `mortelos/agent-standards` and package-owned rules. |
| MCP mount | `routes/ai.php` | Host mounts the framework MCP server when operate mode is enabled. |

Extend a package from a host service provider through its declared outlets: a contract, registry, event, config file or view outlet. Run `php artisan mortelos:outlets:list <package>` to find them; [Outlets](/docs/0/outlets) explains how to use them and when to open a package pull request. `OutletsGuard` checks that your app uses these routes instead of patching or copying package code.

## Expected portal slice output

A complete first vertical slice usually adds:

1. `docs/portals/<slug>/capability-map.md`.
2. A package decision in `.mortelos/package-decisions.md`.
3. Actions for write paths, each returning an `ActionResult`.
4. Aggregates and stored events for auditable facts.
5. A projector and read model for the surface.
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
