---
title: "MortelOS Documentation"
nav_title: "Overview"
slug: "index"
version: "0"
description: "Start building governed Laravel portals with MortelOS."
section: "getting-started"
order: 0
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/index"
last_verified: "2026-06-07"
public: true
---

# MortelOS Documentation

MortelOS is an operating system model for governed Laravel portals. It gives every feature a clear place in a workspace with roles, policies, workflows, audit history and safe agent access.

Start with a host app from `mortelos/starter`, then describe the capabilities your portal needs. A capability is something a role can do, such as uploading a document, approving a request or viewing a dossier.

MortelOS keeps the build order strict:

1. Understand the roles and capabilities.
2. Record package decisions.
3. Model the domain objects and events.
4. Build projections, workflows and surfaces.
5. Verify policies, audit behavior and release evidence.

Use these entry points:

| Page | Use when |
| --- | --- |
| [Installation](/docs/0/installation) | You need to create and verify a fresh host app. |
| [First portal](/docs/0/first-portal) | You need to choose the first useful vertical slice. |
| [Building portals](/docs/0/building-portals) | You need the full capability-first build method. |
| [Host app anatomy](/docs/0/host-app-anatomy) | You need to know where portal code belongs. |
| [TALL conventions](/docs/0/tall-conventions) | You need frontend and Laravel implementation rules. |
| [MCP runtime](/docs/0/mcp-runtime) | You need to understand operate mode and agent access. |
| [Troubleshooting](/docs/0/troubleshooting) | You need fixes for common install and boot failures. |

The current docs version is `0`. The version maps directly to the Git branch named `0` in `mortelos/docs`.
