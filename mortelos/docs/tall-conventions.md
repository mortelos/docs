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
last_verified: "2026-10-14"
public: true
---

# TALL Conventions

MortelOS Starter uses Laravel 13, Livewire 4 single-file components, `mortelos/ui` (built on Flux), Tailwind and Pest.

## Rules

| Rule | Why |
| --- | --- |
| Use `x-mortel::` components from `mortelos/ui` first; check its `COMPONENTS.md` catalogue. Use Flux (`flux:`) only when no matching component exists in `mortelos/ui`. Never write custom HTML for something it already provides. | `mortelos/ui` is the design system. |
| Use Livewire 4 single-file components for new portal surfaces. | State, lifecycle and view stay together. |
| Put every write in an action class under `app/Actions/<Domain>/`: `final`, a `handle()` method, a `public const ABILITY`, a `Mortel\Support\Actor` in and a `Mortel\Actions\ActionResult` out (with a `Mortel\Actions\Undo` when the change can be reversed). | Domain rules stay out of components, and every screen, agent and bridge gets the same result shape. |
| Use Pest for tests. | The starter baseline and capability tests run through Pest. |
| Use Pint for formatting. | One style across host and packages. |
| Deny by default for policies. | Governance remains consistent across web, widgets and agents. |

## Livewire surface shape

Host-owned pages, components and layouts live under:

```text
resources/views/
  pages/<domain>/⚡<name>.blade.php        pages::<domain>.<name>
  components/<group>/⚡<name>.blade.php    shared components
  layouts/app.blade.php, guest.blade.php   layouts::app, layouts::guest
```

A page is a single-file component such as `resources/views/pages/users/⚡index.blade.php`, routed in `routes/starter.php`:

```php
Route::livewire('/users', 'pages::users.index')->name('users');
```

A page picks its layout with `#[Layout('layouts::app')]`; the login page uses `layouts::guest`. A page above about 300 lines splits into components, not into traits.

Package-owned surfaces use the package's Livewire namespace and should not hardcode host-specific classes. Use config and resolver contracts for host-specific behavior.

## The three `Route::livewire` forms

| Form | Use for | Example |
| --- | --- | --- |
| `pages::<domain>.<name>` | A page the host owns. | `Route::livewire('/users', 'pages::users.index')` |
| `layouts::<name>` | A layout, in `#[Layout(...)]` or `<x-layouts::guest>`. | `layouts::app` |
| `<package>::<name>` | A page a package owns. | `overviews::…` |

Do not add `resources/views/livewire/`, classes or traits under `app/Livewire/`, or `Livewire::addNamespace()` for the host's own views. `tests/Feature/ArchGuardTest.php` fails on the first two. The third is a rule in `AGENTS.md`, reviewed by hand.

## A host fact, end to end

A fact the host must be able to audit or undo goes through one chain. The invitations in the starter show it:

| Step | Starter file | Job |
| --- | --- | --- |
| Action | `app/Actions/Invitations/InviteUser.php` | Checks the ability, validates, fills `EventContext` and returns an `ActionResult` with an `Undo`. |
| Aggregate | `app/Aggregates/InvitationAggregate.php` | Holds the guards and calls `recordThat()`. |
| Stored event | `app/Events/Invitations/InvitationCreated.php` | `storedEventType()` returns `invitations.invitation.created`. |
| Event schema | `app/Events/Invitations/invitation.created.v1.schema.json` | Validated when the event is stored; registered in `config/mortel.php`. |
| Event map | `config/event-sourcing.php` | `event_class_map` maps the type to the class, so a renamed class does not break a replay. |
| Projector | `app/Projectors/InvitationProjector.php` | The only writer of the `invitations` table. |
| Read model | `app/Models/Invitation.php` | Listed in `governed_models` in `config/spine.php`. |
| Screen | `resources/views/pages/users/⚡index.blade.php` | Calls the Action, shows the message and the undo button from the `ActionResult`. |

Rules that keep the chain honest:

- Record a stored event only through `recordThat()` in an aggregate. Never `event(new …)` it from an Action, a component or a controller.
- Write the read model only from the projector. Governed models and their allowed writers are listed in `config/spine.php`.
- The read table must be reproducible from the events alone. A host projection is rebuilt with Spatie's `php artisan event-sourcing:replay "App\\Projectors\\InvitationProjector"` (signature `event-sourcing:replay {projector?*}`). `php artisan mortel:projection:rebuild` only covers the framework projections (entity, workflows, workflow_runs, metrics, experiments, transcripts), not host projectors. `tests/Feature/InvitationsTest.php` proves reproducibility with `Spatie\EventSourcing\Projectionist::replay()`: the replayed rows equal the rows before.
- Give the host ability its policy in `PolicyScope::Action`. The starter lists its abilities in `StarterGovernanceGate::SCOPES` in `app/Access/StarterGovernanceGate.php`.

## Domain boundaries

Do not embed domain rules in Blade or Livewire components. Components should:

1. Read projection or service state.
2. Render `x-mortel::` components from `mortelos/ui`.
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

## Done means

Host work is done when these pass:

| Command | Proves |
| --- | --- |
| `composer test:arch` | `ArchGuardTest` and `UiGuardTest` report nothing: Action shape, writes in Actions, events in aggregates, Livewire on the Livewire 4 paths, UI from `<x-mortel::…>` components. |
| `composer test` | The Feature and Unit suites pass, including the replay and the policy tests. |
| `composer analyse` | PHPStan passes, including the governed-model rule from `config/spine.php`. |
| `composer test:browser` | Every screen you touched has a browser test that passes. |
