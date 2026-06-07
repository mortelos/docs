---
title: "Building Portals"
nav_title: "Building portals"
slug: "building-portals"
version: "0"
description: "The capability-first build method for MortelOS portal work."
section: "build-method"
order: 31
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/building-portals"
last_verified: "2026-06-07"
public: true
---

# Building Portals

MortelOS portal work starts with capabilities, not pages. A capability is something a role can do safely inside the workspace.

```text
Customer or operator asks for a portal
  |
  v
Capability-first interview
  |
  v
Capability map
  |
  v
Package decisions
  |
  v
Domain model
  |
  v
Projections and connector boundaries
  |
  v
Surfaces and widgets
  |
  v
Policies and governance
  |
  v
Workflows, inbox and audit
  |
  v
Tests, verification and release notes
```

Do not write implementation code until the capability map and package decisions are clear enough to avoid rework.

## Capability interview

Ask practical questions one at a time:

| Dimension | Question |
| --- | --- |
| Goal | What outcome must the portal make possible? |
| Roles | Who uses the portal and what role do they have? |
| Reads | What may each role view or search? |
| Writes | What may each role create, upload, edit, approve or trigger? |
| Data | Which domain objects matter? |
| External systems | Which systems provide or receive data? |
| Approvals | Which actions need human review? |
| Risk | Which data or action is sensitive? |
| Surfaces | Where should the capability appear? |
| Reuse | Could this serve another MortelOS installation? |

Capture assumptions explicitly. Unknowns that do not block the first slice can go into the plan as follow-ups.

## Package decisions

Every new surface, connector, widget, workflow or reusable service starts with a package decision.

| Decision | Use when |
| --- | --- |
| `package-now` | The capability is reusable across MortelOS installations today. |
| `package-ready` | The host needs local wiring first, but the package boundary is explicit. |
| `workspace-only` | The behavior is specific to one customer or workspace. |

Default to `package-ready` when unsure.

## MortelOS primitives

Map customer language to MortelOS primitives:

| Customer concept | MortelOS primitive | Example |
| --- | --- | --- |
| Customer, project, dossier, document | Entity | `Document` |
| Customer owns dossier | Entity link | `customer -> owns -> dossier` |
| User uploads document | Event | `DocumentUploaded` |
| Pending document list | Projection | `DocumentReviewStatus` |
| Role can approve document | Policy | `document.approve` |
| Review process | Workflow | Pending -> approved or rejected |
| Human review task | Inbox item | `document_review` |

Domain behavior belongs in actions, projections, policies, workflows or package services. Blade and Livewire components read state and call actions.

## Projections

Use a projection when:

1. The screen reads from multiple sources.
2. The data must be explainable later.
3. The same status appears in several surfaces.
4. The state can be rebuilt from canonical facts.
5. Runtime agents need a stable read surface.

Projection rows should store enough source references to explain where data came from. Tests should cover live writes and rebuilds.

## Connectors

A connector is the boundary around an external system. It should usually own provider identity, credential handling, sync jobs, health state, retry behavior, reauth behavior and policy hooks.

Do not put external API calls inside Livewire components. Components consume connector state through actions, projections or package services.

## Surfaces

Pick the smallest surface that fits the workflow.

| Surface | Use for | Typical owner |
| --- | --- | --- |
| Starter page | Core shell page such as dashboard, inbox, users or settings. | Starter host |
| Package route | Reusable feature workspace. | Feature package |
| Dashboard widget | Dense status, metric or queue summary. | Host or package |
| Page widget | Reusable block embedded in a portal page. | Host or package |
| Chat widget | Guided task inside `mortelos/chat`. | Package |
| Inbox item | Human approval or review point. | Host or workflow package |

Use Flux UI before custom Tailwind or Alpine. Use Livewire 4 single-file components for new UI.

## Policies and workflows

Policies decide who can see or do something across web UI, widgets, workflows and agent tools.

Rules:

1. Deny by default.
2. Seed safe defaults during onboarding.
3. Give every mutating action a policy ability.
4. Give every mutating agent tool a policy ability.
5. Route policy changes through proposal and approval flows.

Use inbox or proposal flows for document reviews, customer approvals, connector setup, connector reauth, AI-proposed changes, policy changes and risky data mutations.

## Build plan

After the capability map and package decisions, write a build plan with:

1. Goal.
2. Roles.
3. Capability map summary.
4. Package decisions.
5. Domain model.
6. Projections.
7. Connectors.
8. Surfaces.
9. Policies.
10. Workflows and inbox.
11. Tenant identity.
12. Observability.
13. Test plan.
14. Release notes and rollback.
15. First vertical slice.

Use `N/A, reason: ...` for sections that genuinely do not apply. Do not leave `TBD` unless the user accepted that unknown as a blocker.

## Verification

Before claiming a portal change is done:

```bash
vendor/bin/pint --dirty
php artisan starter:doctor
vendor/bin/pest
```

Also run a manual smoke test from login to dashboard and through the new capability surface. For approvals, verify both approve and reject paths.
