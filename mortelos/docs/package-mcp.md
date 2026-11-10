---
title: "MortelOS MCP"
nav_title: "MCP"
slug: "package-mcp"
version: "0"
description: "Lets an employee connect an AI client such as Claude Code or Codex to a MortelOS app with their own login, with a connect window and a list of connected apps."
section: "packages"
order: 68
status: "mvp"
audience: "developers"
package: "mortelos/mcp"
canonical_path: "/docs/0/package-mcp"
last_verified: "2026-11-10"
public: true
---

# MortelOS MCP

`mortelos/mcp` lets an employee connect an AI client, such as Claude Code or Codex CLI, to your app with their own app login. The client registers itself over OAuth (Laravel Passport and `laravel/mcp`), the employee approves it on a consent screen, and from then on the client sees only the tools the employee's role may also use in the chat. This page describes version 0.2.0. For the boundaries of the MCP runtime in general, read [MCP runtime](/docs/0/mcp-runtime).

## What It Does And Does Not Do

| Does | Does not |
| --- | --- |
| Mounts the MCP server (and any server you add) behind OAuth login, the `mcp:use` scope, throttling and your tenancy middleware. | Offer write tools or approvals over MCP. A tool that needs approval is never offered, and a call that would need one is refused. |
| Offers only agent tools that are read-only, need no approval and are allowed for the user's role. Rights are checked again on every call. | Bind tokens to a tenant. Tokens are Passport tokens; tenant binding is outside this package. |
| Ships the connect window `mortelos-mcp::connect` and the list `mortelos-mcp::connections`. | Provide a login screen. Login, tenancy, Passport migrations and signing keys stay with the host app. |
| Records every tool call as an agent run with source `mcp`, and every disconnect as the stored event `mcp.connection.revoked`. | Restore a token by replaying events. Reconnecting is the way back. |

## Requirements

| | |
| --- | --- |
| PHP | `^8.4` |
| `mortelos/framework` | `^0.7.60 \|\| ^0.8` |
| `mortelos/ui` | `^0.6.19` |
| `laravel/passport` | `^13.7` |
| `laravel/mcp` | `^1.1` |
| `livewire/livewire` | `^4.0` |

The app also needs the agent runtime tables of `mortelos/framework` (`agent_runs` and `agent_run_steps`, with the later migrations that add columns to them). An app that already uses chat or agent runs has them. Without them every tool call fails.

## Worked Example: An App With Its Own Rights Per Role

This example follows an app that does not use MortelOS action policies for its rights, but has its own roles and its own check ("may this role connect an AI assistant?"). It goes from install to a connected client.

### 1. Install And Switch It On

```bash
composer config repositories.mortelos composer https://packages.mortelos.com   # once, if the app has not got it yet
composer require mortelos/mcp
php artisan vendor:publish --tag=passport-migrations
php artisan migrate
php artisan passport:keys
```

On a server where `storage/` is not shared, set `PASSPORT_PRIVATE_KEY` and `PASSPORT_PUBLIC_KEY` instead of running `passport:keys`. The user model implements `Laravel\Passport\Contracts\OAuthenticatable` and uses `Laravel\Passport\HasApiTokens`. A multi-tenant app sets `PASSPORT_CONNECTION` to the central connection and runs the Passport migrations there. If users have string or ULID ids, change `user_id` in `oauth_access_tokens`, `oauth_auth_codes` and `oauth_device_codes` to a string before the first token is issued.

MCP stays off until you switch it on:

```dotenv
MORTELOS_MCP_ENABLED=true
```

Publish the config when you need to change more than that:

```bash
php artisan vendor:publish --tag=mortelos-mcp-config
```

A multi-tenant app puts its tenancy middleware in `middleware` of `config/mortelos-mcp.php`. That list runs after authentication, the scope check and throttling, and before the API context is resolved.

If the app caches its routes, run `php artisan route:cache` again after switching MCP on, and also after switching it off: routes are registered when the cache is built.

### 2. Connect The App's Own Rights

By default the package decides access with the `mcp.connect` and `mcp.manage` abilities as `action` policies on the user's role. An app with its own rights model binds the contract `Mortelos\Mcp\Contracts\McpAccessGate` instead:

```php
// app/Providers/AppServiceProvider.php
public function register(): void
{
    $this->app->bind(
        \Mortelos\Mcp\Contracts\McpAccessGate::class,
        \App\Access\AppMcpAccess::class,
    );
}
```

```php
namespace App\Access;

use App\Models\User;
use Mortel\Support\Actor;
use Mortelos\Mcp\Contracts\McpAccessGate;

final class AppMcpAccess implements McpAccessGate
{
    public function allows(Actor $actor, string $ability): bool
    {
        if ($actor->isSystem()) {
            return true;
        }

        if (! $actor->isUser() || $actor->userId() === null) {
            return false;
        }

        $user = User::query()->find($actor->userId());

        if ($user === null || $user->isBlocked()) {
            return false;
        }

        return match ($ability) {
            'mcp.connect' => $user->role?->may('assistant.connect') ?? false,
            'mcp.manage' => $user->role?->may('assistant.manage') ?? false,
            default => false,
        };
    }
}
```

The class must deny by default in the current tenant. The binding covers MCP entry (`initialize`, `tools/list`, `tools/call`), the Toestaan button on the consent screen (`mcp.connect`), the connect window, the list of connected apps and disconnecting the apps of other users (`mcp.manage`). A user's own connections and the system console keep their existing disconnect rules. The package registers its default binding even when MCP is switched off and never overrides a binding your app made.

Which tools a role gets still follows from the `agent_tool` policies of that role, the same ones the chat uses. `tools.allowlist` can narrow that further.

### 3. Place The Connect Window And The List

Put both components on one settings page, for example "Connected apps". Route the page behind `web` and `auth`:

```php
Route::middleware(['web', 'auth'])->group(function (): void {
    Route::view('/settings/connected-apps', 'settings.connected-apps');
});
```

```blade
{{-- resources/views/settings/connected-apps.blade.php --}}
<livewire:mortelos-mcp::connections />

<livewire:mortelos-mcp::connect
    name="acme"
    help="Nothing opens? Switch the VPN on."
    hint="Is the VPN on?"
    example="What does Acme say about returns?" />
```

The list has its own **Koppelen** button. To add a button of your own elsewhere on the page, dispatch the same browser event:

```blade
<x-mortel::button variant="primary" x-on:click="$dispatch('mortelos-mcp:connect')">Koppelen</x-mortel::button>
```

Properties of `mortelos-mcp::connect` (all optional):

| Property | Default | Meaning |
| --- | --- | --- |
| `server` | The first key of `servers`. | Which server the address points to. |
| `name` | The app name as a slug. | The name the commands register the server under. Always slugged. |
| `help` | "Opent er niets? Vraag je beheerder om hulp." | One sentence behind the info icon at step 2. Put VPN or certificate advice here. |
| `hint` | "Is de inlogpagina geopend?" | Text after "Nog niets binnen." when nothing arrived after two minutes. |
| `example` | None. | An example question shown after success. |

`mortelos-mcp::connections` takes `:heading="false"` when your page has its own header.

The window does nothing for a user the gate refuses, or while MCP is off, so a button may stay behind the condition you already use. The window opens on the browser event `mortelos-mcp:connect` (its `open()` action does the same) and sends `mortelos-mcp:connected` on `window` when the user clicks Klaar or closes it after success. The list refreshes itself on that event. The success view has the outlet `mcp.connect.connected`, which shows the example question by default. Programs and their order follow `connect.clients` (`claude-code`, `codex`, `other`).

The package ships Dutch and English language files, and the window follows the app locale. The texts quoted on this page are the Dutch ones.

### 4. What The Employee Sees

An employee whose role is allowed opens the page "Connected apps".

1. With no connection yet, the list says "Nog geen apps gekoppeld" and shows a **Koppelen** button. With connections, the button sits next to the heading "Gekoppelde apps".
2. **Koppelen** opens the window "AI-assistent koppelen". It offers three programs: Claude Code, Codex and Ander. It starts on the first program the employee has not connected yet. Ander shows only the address.
3. The window shows three numbered steps. Step 1 is "Plak dit in je terminal" with a ready-made command to copy, including the address of this environment. Step 2 depends on the program. For Claude Code it says to type `/mcp` in Claude Code, pick the app and choose Authenticate. For Codex it says to sign in in the browser that opens. Next to step 2 an info icon shows your `help` sentence. Step 3 is clicking **Toestaan** on the consent screen (for Claude Code it also says to sign in first).
4. Below the steps a line says "Wacht tot je op Toestaan klikt". After two minutes without a result it adds "Nog niets binnen." and your `hint`.
5. When the employee clicks **Toestaan**, the window switches by itself to "Gekoppeld met Codex" (or whichever program), shows "Vraag het nu in Codex, bijvoorbeeld:" with your `example` question, and a **Klaar** button.
6. The list now shows the app with a small mark, when it was last renewed and when it expires ("vandaag 12:01", "gisteren 09:15", "10 dec"). Ontkoppelen asks first and then says which app it was ("Codex is ontkoppeld."). An administrator with the manage right sees all employees' connections, with the employee's name, under the subtitle "Van alle medewerkers".

How the window knows it worked: when it opens it remembers which clients the user already has an active token for. While it waits it checks every 3 seconds and reports success only for a client that was not in that set, so a renewed token of an already connected client does not count.

From a shell, across all tenants:

```bash
php artisan mortelos-mcp:connections                      # list active connections
php artisan mortelos-mcp:connections --user=<user-id>     # one user
php artisan mortelos-mcp:connections --user=<user-id> --revoke=<client-id>
```

## Connecting A Client

Use the same address everywhere: an IP or a host name, not both. The MCP endpoint is `https://<host>/mcp/mortelos`, the `path` of the default server. The window shows this address, built from `APP_URL` plus the `path`.

### Codex CLI

```bash
codex mcp add mortelos --url https://<host>/mcp/mortelos
codex mcp login mortelos
```

The browser opens the app's login and then the consent screen. On a machine without a browser, run `codex mcp login mortelos --no-browser`.

### Claude Code

```bash
claude mcp add --transport http mortelos https://<host>/mcp/mortelos
```

Then run `/mcp` inside Claude Code, pick the server and choose Authenticate.

## Pitfalls

| Pitfall | What to do |
| --- | --- |
| Two addresses for one app (an IP in one place, a host name in another). | Use one address everywhere. Codex requires the issuer, authorize and token endpoints on one origin, so `APP_URL` must equal the address the client uses. |
| A self-signed certificate. | Codex: set `CODEX_CA_CERTIFICATE=/path/to/cert.pem` (or `SSL_CERT_FILE`); the certificate needs the address as a subject alternative name (SAN) and must be `CA:FALSE`. A certificate with only a CN, or with `CA:TRUE`, is refused. Check with `openssl x509 -noout -ext subjectAltName,basicConstraints -in cert.pem`. Claude Code: set `NODE_EXTRA_CA_CERTS=/path/to/cert.pem`. The employee's browser must trust it too. |
| Routes are cached. | Run `php artisan route:cache` again after switching MCP on or off. |
| Every MCP request gets 403, `initialize` included. | The user lacks the connect right: check `mcp.connect` in your gate or policy. |
| The app has other Passport clients (its own apps, tools or bots that sign in through `/oauth/*`). | Set `passport_routes_when_disabled` to `true`. While MCP is off the package otherwise calls `Passport::ignoreRoutes()`, and installing it would take those routes away. It has no effect while MCP is on. |
| Passport is app-wide. | The consent view and the token lifetimes are set on Passport itself, so they apply to every Passport client. Set `consent.view` and the `tokens.*` keys to `null` to keep your own. |
| The administrator view shows nothing in an app without `tenant_user`. | That view reads `tenant_user` on the central connection directly. The shell command does not need it. |

## Configuration

`config/mortelos-mcp.php` is platform configuration, read from config only and never from tenant settings.

| Key | Default | Meaning |
| --- | --- | --- |
| `enabled` | `env('MORTELOS_MCP_ENABLED', false)` | Registers routes and the Passport integration. |
| `passport_routes_when_disabled` | `false` | Keeps Passport's own `/oauth/*` routes while MCP is off. |
| `servers` | `mortelos` with path `mcp/mortelos` and `AgentToolsServer` | Handle to `path`, `server` and optional `middleware`. Each entry is mounted behind the full access stack. |
| `middleware` | `[]` | Host middleware for all servers, such as tenancy. |
| `redirect_domains` | `['http://127.0.0.1']` | Allowed redirect domains. Loopback on any port is accepted by default; private-use schemes stay the `mcp.custom_schemes` setting of `laravel/mcp`. |
| `tools.allowlist` | `null` | A list of tool names or `fnmatch` patterns that limits what may leave through MCP. |
| `tokens.access_ttl_minutes` | `60` | Access token lifetime. `null` keeps Passport's. |
| `tokens.refresh_ttl_days` | `30` | Refresh token lifetime. `null` keeps Passport's. |
| `throttle.oauth` | `'20,1'` | Discovery and client registration, per IP. |
| `throttle.mcp` | `'120,1'` | Authenticated MCP requests. |
| `connect.clients` | `['claude-code', 'codex', 'other']` | Programs in the connect window, in this order. |
| `consent.view` | `'mortelos-mcp::authorize'` | The consent screen. `null` keeps the host's view. |
| `consent.vite` | `['resources/css/app.css']` | Vite entry points of the consent screen. |

When the app already has a server of its own to offer, add an entry to `servers` with its own `middleware` instead of calling `Mcp::web()` yourself. The read-only guarantee belongs to the package's `AgentToolsServer`; a server you add brings its own tools, so check which of them can change data.

## Outlets

The table lists the outlets in `resources/mortelos/outlets.json` as of version 0.2.0; read [Outlets](/docs/0/outlets) for how each kind works.

```bash
php artisan mortelos:outlets:list mcp
```

| Kind | Name | What it is for |
| --- | --- | --- |
| contract | `Mortelos\Mcp\Contracts\McpAccessGate` | Decides connect and manage access for the current actor and tenant. Bind your own deny-by-default rights model. |
| config | `mortelos-mcp` | Servers and their middleware, host middleware, redirect domains, tool allowlist, token lifetimes, throttles, the programs of the connect window and the consent screen. Publish with `php artisan vendor:publish --tag=mortelos-mcp-config`. |
| view | `mcp.connect.connected` | The success view of the connect window, under "Gekoppeld met" and above Klaar. Data: `client`, `example`. |
| event | `Mortelos\Mcp\Events\McpConnectionRevoked` | Stored as `mcp.connection.revoked` when a connection is disconnected. Audit only. |

## Boundaries

Use this package for the OAuth entry, the read-only tool offer, the connect window and the list of connected apps. Login, tenancy, signing keys and the rights model stay in the host app. New runtime tools belong in `mortelos/framework` or a package with explicit contracts, not in `mortelos/starter`.
