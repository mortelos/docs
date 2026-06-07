---
title: "Agentic Development"
nav_title: "Agentic development"
slug: "agentic-development"
version: "0"
description: "Use an AI coding agent to build MortelOS portals without skipping governance."
section: "ai"
order: 30
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/agentic-development"
last_verified: "2026-06-07"
public: true
---

# Agentic Development

MortelOS is designed for human-directed, agent-assisted portal builds.

The user owns product judgment, scope and acceptance. The agent wires routes, actions, policies, projections, tests and package boundaries.

There are two AI modes:

| Mode | Purpose | Boundary |
| --- | --- | --- |
| Build mode | A coding agent assembles the host app and packages. | Files, Artisan commands, Composer, tests and review. |
| Operate mode | A runtime agent operates a live workspace. | OAuth-scoped MCP tools from `mortelos/framework`. |

Use build mode while creating the portal. Use operate mode after the workspace is running and policy-governed.

## Recommended prompt

```text
Use the setup-portal skill.
I want to build a customer portal for: [describe the customer, user group or process].
```

If your agent does not support local skills, start with a capability-first interview:

```text
You are working in a MortelOS Starter host app.

Read AGENTS.md, README.md and docs/building-portals.md first.
If the setup-portal skill is available, use it now.
Use a capability-first interview before writing code.

I want to build a customer portal for: [describe the customer, user group or process].

Ask focused questions until the capability map is clear:
1. Which roles use the portal?
2. What can each role view, upload, approve or trigger?
3. Which data is shown or changed?
4. Which external systems are involved?
5. Which actions need human approval?
6. Which parts should become reusable MortelOS packages?

After the interview, write a build plan before implementation.
Use MortelOS primitives: entities, entity links, events, projections,
connectors, policies, workflows, inbox items and surfaces.
Record package decisions before adding new surfaces.
Use deny-by-default policies for mutating actions and sensitive data.
Stop for review before building the first vertical slice.
```

## Operating rule

The agent can move quickly after the capability map is accepted. Before that point, speed creates rework.

## Agent checklist

1. Read `AGENTS.md`.
2. Read `README.md`.
3. Read `docs/building-portals.md`.
4. Check existing portal docs under `docs/portals/`.
5. Interview before planning.
6. Write or update the capability map.
7. Record package decisions.
8. Write the build plan.
9. Build one vertical slice.
10. Verify with doctor, tests and manual smoke.
11. Stop for user review.

Do not invent a tenant model, put domain rules in Blade or Livewire components, add a surface without a package decision, bypass policies with local UI checks, or claim success without verification.
