---
title: "MCP Runtime"
nav_title: "MCP runtime"
slug: "mcp-runtime"
version: "0"
description: "Operate-mode MCP runtime boundaries for MortelOS host apps."
section: "ai"
order: 34
status: "mvp"
audience: "developers"
package: "mortelos/mcp"
canonical_path: "/docs/0/mcp-runtime"
last_verified: "2026-11-06"
public: true
---

# MCP Runtime

MortelOS has two AI surfaces:

| Mode | Purpose | Tools |
| --- | --- | --- |
| Build mode | A coding agent assembles the host app from primitives. | File edits, Artisan, Composer, tests and the `setup-portal` skill. |
| Operate mode | A runtime agent operates the live workspace. | OAuth-authenticated MCP tools served from the host. |

The MCP server is not used for portal bootstrap. Bootstrap is code, config, Artisan and tests. MCP starts once the workspace is live.

## Host mount

`mortelos/mcp` is the standard way to mount MCP in a host. It puts the MCP server behind OAuth login (Laravel Passport and `laravel/mcp`), so an employee connects an AI client such as Codex CLI or Claude Code with their own app login.

```bash
composer require mortelos/mcp
```

```dotenv
MORTELOS_MCP_ENABLED=true
```

The package is off until you set that variable. When it is on, the package mounts everything itself, so the host adds no route to `routes/ai.php`:

- `Mcp::oauthRoutes()` for discovery and client registration.
- The MCP server at `/mcp/mortelos`, behind authentication, the `mcp:use` scope and throttling.
- Access only for users whose role has the `mcp.connect` right. Without it every request gets 403.
- Only agent tools that are read-only, need no approval and that the user's role may also use in the chat. Rights are checked again on every call.

Keep the route stable as `/mcp/mortelos`. Customer-specific MCP routes belong only in that host's own documentation.

The package also needs Passport migrations and signing keys, a user model that implements `OAuthenticatable`, and the `mcp.connect` policy for the right roles. The full host setup, the settings in `config/mortelos-mcp.php` and the steps to connect a client are in the package README, `vendor/mortelos/mcp/README.md`. A worked example, from install to a connected client and including the connect window and the list of connected apps, is on the page [MortelOS MCP](/docs/0/package-mcp).

### Mount a server of your own

The package config key `servers` maps a handle to a `path` and a `Laravel\Mcp\Server` class. Each entry is mounted behind the same access stack as the default server: authentication, the `mcp:use` scope, throttling, host middleware and the `mcp.connect` check. A host that wants an extra server adds an entry there instead of calling `Mcp::web()` itself.

The read-only guarantee above belongs to the package's own `AgentToolsServer`. A server you add brings its own tools, so check which of them can change data before you expose it.

The framework's `Mortel\MCP\Servers\UteqOSServer` has write and governance tools and is not part of the package's offer. A host that wants it mounts it by hand with `Mcp::oauthRoutes()` and `Mcp::web('/mcp/mortelos', UteqOSServer::class)` in `routes/ai.php`, without `mortelos/mcp`, and then supplies its own authentication (`auth:api`), tenancy, role resolution, trust-level enforcement, data classification and throttling in the middleware.

## What MCP exposes

The framework owns the operating-system level tool surface: entities, links, policies, workflows, inbox, skills, agent runs and context. Channel packages expose external data through connector actions and package contracts, not by changing the host route.

Every mutating MCP action needs:

1. A policy ability.
2. Tenant scoping.
3. Actor resolution.
4. Audit or event evidence.
5. A clear failure mode when approval or runtime setup is missing.

## Security rules

| Concern | Rule |
| --- | --- |
| Authentication | OAuth 2.1 with dynamic client registration. |
| Tenancy | Tenant context is initialized before framework tools execute. |
| Authorization | The same policies govern web UI, widgets and MCP tools. |
| Trust | Sensitive or mutating actions require the configured trust level. |
| Classification | Data classification applies before agent-visible output. |
| Throttling | Runtime tools should be rate limited per tenant and actor. |

Do not expose new MCP tools from inside `mortelos/starter`. Starter is the host template. Reusable runtime behavior belongs in `mortelos/framework` or a package with explicit contracts.
