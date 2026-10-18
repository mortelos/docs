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
package: "mortelos/framework"
canonical_path: "/docs/0/mcp-runtime"
last_verified: "2026-10-18"
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

A host mounts the MortelOS MCP server from `mortelos/framework` in `routes/ai.php`:

```php
use Laravel\Mcp\Facades\Mcp;
use Mortel\MCP\Servers\UteqOSServer;

Mcp::oauthRoutes();

Mcp::web('/mcp/mortelos', UteqOSServer::class)
    ->middleware([
        'auth:api',
        // tenancy init from the MCP token
        // role resolution
        // trust-level enforcement
        // data classification
        // throttling
    ]);
```

Keep the route stable as `/mcp/mortelos`. Customer-specific MCP routes belong only in that host's own documentation.

### The Mortel API server

From `mortelos/framework` 0.7.7 a host can mount a second server, `Mortel\MCP\Servers\MortelServer`, beside the first one. It needs `ResolveApiContext` after the host's own auth and tenancy middleware:

```php
use Laravel\Mcp\Facades\Mcp;
use Mortel\Api\Http\Middleware\ResolveApiContext;
use Mortel\MCP\Servers\MortelServer;

Mcp::web('/mcp/mortel', MortelServer::class)->middleware([
    'auth:api',
    InitializeTenancyByRequestData::class, // or your own tenancy middleware
    ResolveApiContext::class,
]);
```

`UteqOSServer` has a fixed set of tools. `MortelServer` has none of its own: it builds one tool per entry in the Mortel API registries, `query_<name>` for each named query and `action_<name>` for each public action, with dots as dashes. A tool the user may not use is not listed. The tools run the same checks and return the same bodies and receipts as the HTTP API, so an action can be retried over either transport without running twice. It does not work without the HTTP middleware above and has no stdio mount. See [Mortel API v1](/docs/0/mortel-api#mcp).

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
