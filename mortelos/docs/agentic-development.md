---
title: "Agentic Development"
nav_title: "Agentic development"
slug: "agentic-development"
version: "0"
description: "Use an AI coding agent to build MortelOS portals without skipping governance."
section: "build-method"
order: 30
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/agentic-development"
last_verified: "2026-05-31"
public: true
---

# Agentic Development

MortelOS is designed for human-directed, agent-assisted portal builds.

The user owns product judgment, scope and acceptance. The agent wires routes, actions, policies, projections, tests and package boundaries.

## Recommended prompt

```text
Use the portal-kickoff skill.
I want to build a customer portal for: [describe the customer, user group or process].
```

If your agent does not support local skills, start with a capability-first interview:

```text
You are working in a MortelOS Starter host app.

Read AGENTS.md, README.md and docs/building-portals.md first.
Use a capability-first interview before writing code.

Ask focused questions until the capability map is clear:
1. Which roles use the portal?
2. What can each role view, upload, approve or trigger?
3. Which data is shown or changed?
4. Which external systems are involved?
5. Which actions need human approval?
6. Which parts should become reusable MortelOS packages?

After the interview, write a build plan before implementation.
Record package decisions before adding new surfaces.
Use deny-by-default policies for mutating actions and sensitive data.
Stop for review before building the first vertical slice.
```

## Operating rule

The agent can move quickly after the capability map is accepted. Before that point, speed creates rework.

