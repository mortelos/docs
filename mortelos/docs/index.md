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
last_verified: "2026-05-31"
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

The current docs version is `0`. The version maps directly to the Git branch named `0` in `mortelos/docs`.

