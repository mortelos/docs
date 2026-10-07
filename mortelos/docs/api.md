---
title: "API"
nav_title: "API"
slug: "api"
version: "0"
description: "What MortelOS exposes to clients outside the host app in v0, and how to build your own front end."
section: "reference"
order: 70
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/api"
last_verified: "2026-10-07"
public: true
---

# API

MortelOS v0 has no general public HTTP API. Neither `mortelos/framework` nor `mortelos/starter` ships REST or JSON routes for entities, the inbox, governance or users. This page lists what a client outside the host app can call today, so you can decide how to build your own front end.

## Can I Build My Own Front End?

Yes, but you build the HTTP layer in the host. A MortelOS portal is a Laravel app. `mortelos/framework` provides actions, services, models and contracts and registers no routes. `mortelos/starter` serves its screens as Livewire pages behind session login, and has no `routes/api.php`, no API token guard and no OAuth server.

| Approach | What you do |
| --- | --- |
| Screens in the host | Add Livewire pages to the host with `mortelos/ui`. This is the supported path; see [Surfaces](/docs/0/building-portals#surfaces). |
| Separate front end | Add API routes and a token guard to the host, and let those controllers call framework actions and services. These routes are host code, not a MortelOS contract, so you version, document and secure them yourself. |
| Agent or MCP client | Mount the framework MCP server in the host and connect over OAuth. See [MCP server](#mcp-server). |

## What Exists Today

### MCP server

`mortelos/framework` ships `Mortel\MCP\Servers\UteqOSServer`, a Laravel MCP server. It is the closest thing to an external API in v0.

| Area | Tools |
| --- | --- |
| Entities | `entity-search-tool`, `entity-get-tool`, `entity-create-tool`, `entity-update-tool`, `entity-link-tool`, `entity-history-tool` |
| Skills and workflows | `skill-run-tool`, `workflow-trigger-tool` |
| Governance | `governance-pending-tool`, `governance-approve-tool`, `governance-reject-tool` |
| Agent runs | `agent-run-tool`, `agent-status-tool`, `agent-cancel-tool` |
| Context | `site-context-tool`, `site-conventions-tool`, `site-register-tool`, `ask-tool` |

The framework does not mount the server. The host mounts it in `routes/ai.php` with `Mcp::oauthRoutes()` and `Mcp::web()` behind the `auth:api` guard, which needs `laravel/passport` in the host. See [Host mount](/docs/0/mcp-runtime#host-mount). The starter does not do this out of the box.

The tools are built for agents: a client calls a named tool with JSON arguments and gets JSON text back. Their output is not documented as a stable schema in v0, so a front end that relies on it should expect changes.

### Entity graph API

`mortelos/entity-graph` registers four read-only JSON routes: `GET /api/v1/entity-graph` and its `/expand`, `/search` and `/path` sub-routes. Their default middleware starts with `auth:api`, the same token guard as the MCP mount, so they only work once the host has one. See [Entity graph](/docs/0/package-entity-graph#use-it-for) for what each route does. Set `entity-graph.api.enabled` to `false` to turn them off.

### Package routes

Feature packages register Livewire pages such as `/chat`, `/documents` and `/planner`. Channel packages register webhook and OAuth callback routes for their external system. Neither is a client API.

## Not There Yet

- A versioned REST or JSON API for entities, inbox, governance, users or settings.
- API tokens or an OAuth server in the starter.
- A published reference for MCP tool input and output.

This page becomes the API reference once those contracts are verified against released source.
