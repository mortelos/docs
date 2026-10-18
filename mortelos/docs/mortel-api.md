---
title: "Mortel API v1"
nav_title: "Mortel API"
slug: "mortel-api"
version: "0"
description: "Switch on the versioned Mortel API in a host app: named queries, public actions with receipts and live change notifications over HTTP and MCP."
section: "reference"
order: 72
status: "mvp"
audience: "developers"
package: "mortelos/framework"
canonical_path: "/docs/0/mortel-api"
last_verified: "2026-10-18"
public: true
---

# Mortel API v1

The Mortel API is one versioned contract per installation. A client uses it to read named queries, to run public actions and get a receipt back, and to hear when something changes. It works over HTTP and over MCP, with the same checks on both. It ships in `mortelos/framework` 0.7.7 and 0.8.3.

The API works per brain. A brain is one tenant of the installation, with its own members, policies and data. Every call runs in the brain that the host's tenancy middleware selected.

## Switching it on

The API is off. The framework registers no route, channel or MCP server until the host does the steps below.

1. Mount the routes in `routes/api.php`, inside the host's own auth and tenancy middleware. `auth:api` needs a token guard named `api`, for example from `laravel/passport`. Use the tenancy middleware of your host; the example uses the one from `stancl/tenancy`.

   ```php
   use Mortel\Api\MortelApi;

   Route::middleware(['auth:api', InitializeTenancyByRequestData::class])
       ->group(fn () => MortelApi::routes());
   ```

   Laravel prefixes `routes/api.php` with `api`, so every address starts with `/api/mortel/v1`. Route names start with `mortel.api.v1.`. `MortelApi::routes()` adds the `ResolveApiContext` middleware itself.

2. Publish the framework migrations and run them for every brain. The `api_operations` table stores receipts and idempotency keys, and it belongs in each brain's database, not in the central one.

   ```bash
   php artisan vendor:publish --tag=os-core-migrations
   php artisan migrate            # or tenants:migrate, where your framework migrations live
   ```

3. For live notifications, list the projector in `config/event-sourcing.php`, register the channel in `routes/channels.php` and run a broadcaster such as Laravel Reverb with a queue worker. See [Live notifications](#live-notifications).

   ```php
   'projectors' => [
       // ...
       Mortel\Api\Live\ChangeNotifier::class,
   ],
   ```

   ```php
   Mortel\Api\MortelApi::channels();
   ```

4. For MCP clients, mount the second MCP server. See [MCP](#mcp).

When the host's auth middleware refuses a request, it answers in its own error shape. If you want the Mortel error shape for a 401 too, leave `auth:api` out of the group and set `MORTEL_API_GUARD=api` (config key `mortel.api.guard`). The API then reads the user from that guard and answers 401 `unauthenticated` itself.

Every request needs a user with a role in the current brain. No user gives 401 `unauthenticated`. A user who is not a member, or whose role does not resolve, gets 403 `forbidden`.

## Routes

| Method | Address | What it returns |
| --- | --- | --- |
| GET | `/api/mortel/v1` | Discovery: the version, the brain, the queries and actions this user may use. |
| GET | `/api/mortel/v1/queries/{query}` | One page of a named query. |
| POST | `/api/mortel/v1/actions/{action}` | A receipt for a public action. |
| GET | `/api/mortel/v1/operations/{operation}` | The stored receipt of an earlier action call. |

Query and action names are lowercase words joined by dots, such as `entities` or `entities.rename`. The framework registers the `entities` query and the `entities.rename` action.

### Discovery

`GET /api/mortel/v1` returns:

```json
{
  "api": {"version": "v1", "supported_versions": ["v1"]},
  "installation": {"name": "Example", "host": "app.example.com"},
  "brain": {"id": "01j...", "name": "Example"},
  "policy_revision": "01k...",
  "queries": [
    {"name": "entities", "description": "...", "denied": false, "item_type": "entities",
     "fields": [{"name": "name", "type": "string", "label": "Name", "classification": "public"}]}
  ],
  "actions": [
    {"name": "entities.rename", "description": "...", "denied": false, "item_type": "entities",
     "input_schema": {"type": "object", "properties": {"name": {"type": "string"}}, "required": ["name"]},
     "queued": false}
  ],
  "live": {"channel": "mortel.brain.01j...", "type": "private"}
}
```

`installation` comes from `config('app.name')` and the host of `config('app.url')`. An entry the user may not use has `denied: true` and no fields or schema. `fields` lists only the fields the user's trust level may see. `live` is there only after the host called `MortelApi::channels()`.

### Queries

`GET /api/mortel/v1/queries/entities?page=1&per_page=50` returns one page. `page` is at least 1 and `per_page` is between 1 and 100 (default 50).

```json
{
  "rows": [{
    "item": {"brain": "01j...", "type": "entities", "ref": "01k...", "revision": "2"},
    "values": {"name": "Example", "type": "customer", "status": "live", "updated_at": "2026-10-17T20:00:00+00:00"},
    "meanings": {"status": {"label": "Live", "tone": "success"}},
    "can": ["entities.rename"]
  }],
  "total": 1,
  "page": 1,
  "per_page": 50
}
```

Rows and `total` only count what the user may read. A field above the user's trust level is absent, not null. `can` names the actions the user may run on that row now; it is a hint, and the action checks again.

Every answer has an `ETag` and `Cache-Control: private, no-cache`. Send the ETag back in `If-None-Match`: an unchanged answer is a 304 with no body. Login, membership and policy are checked before the ETag is compared, so a revoked right gives 401 or 403, never 304.

### Actions

Send the item and revision you got from a query, with an `Idempotency-Key` header:

```http
POST /api/mortel/v1/actions/entities.rename
Idempotency-Key: rename-2026-10-17-1
Content-Type: application/json

{"item": {"brain": "01j...", "type": "entities", "ref": "01k...", "revision": "1"},
 "expected_revision": "1",
 "input": {"name": "New name"}}
```

The answer is a [receipt](#receipts). Send the revision back as you got it; it is an opaque string. An action without an item type has no `item` and no `expected_revision`. A real call without a key gets 400 `idempotency_key_missing`.

The same key with the same call returns the stored receipt and runs nothing twice, so a client can safely retry after a lost response. The same key with a different call gets 422 `idempotency_key_reused`. Keys are scoped to the OAuth client and the user, so a token refresh keeps them.

Send `Precognition: true` with the same body to check the call without running it. No key is needed. Success is a 204 with `Precognition-Success: true`; a refusal has the same status and body as a real call. A Precognition success grants nothing: the real call checks everything again.

An action whose `queued()` is true answers 202 with status `accepted` and runs in a job. Before it runs, the job checks the grant, the membership, the rights, the state and the revision again. A job must start within 120 seconds of acceptance, or the operation ends as `failed` with `operation_expired`.

### Operations

`GET /api/mortel/v1/operations/01k...` returns the stored receipt, with status 200 whatever the receipt says. Poll it after a 202 until the status is no longer `accepted`. Only the same OAuth client and user may read it; anyone else gets 404 `not_found`. When the user lost access to the item, the answer only has `operation_id`, `status`, `correlation_id` and `error`.

## Receipts

```json
{
  "operation_id": "01k...",
  "status": "done",
  "message": "The name was changed.",
  "undo": {"action": "entities.rename", "item": {"brain": "01j...", "type": "entities", "ref": "01k...", "revision": "2"},
           "expected_revision": "2", "input": {"name": "Old name"}},
  "changed": {"queries": ["entities"], "items": [{"brain": "01j...", "type": "entities", "ref": "01k...", "revision": "2"}]},
  "correlation_id": "optional-client-trace",
  "error": null
}
```

| Status | HTTP | Meaning |
| --- | --- | --- |
| `done` | 200 | The action ran. |
| `skipped` | 200 | Nothing needed to change. |
| `approval_needed` | 202 | The action waits for approval. |
| `accepted` | 202 | A queued action that has not run yet. |
| `failed` | 422 | The action failed; `error` says why. |
| `unknown` | 200 | The placeholder of a direct call while it runs, inside its transaction. A client does not normally see it. |

`undo` is a ready call: use its `action` in the address, send the rest as the body, with a new idempotency key. `changed.queries` tells the client which queries to read again. The correlation id comes from the `X-Correlation-Id` header, or is a new ULID.

## Errors

Every error has one shape. The message follows the app locale.

```json
{"error": {"code": "forbidden", "message": "You are not allowed to do this."}}
```

| Code | HTTP | When |
| --- | --- | --- |
| `unauthenticated` | 401 | No user. |
| `forbidden` | 403 | Not a member, or the policy refuses. |
| `not_found` | 404 | Unknown name, or an item the user may not see. |
| `not_allowed_now` | 409 | The item's state does not allow the action now. |
| `revision_conflict` | 409 | The item changed; `error.current_revision` has the new revision. |
| `validation_failed` | 422 | The input does not match the schema; field errors are in `error.errors`. |
| `idempotency_key_missing` | 400 | A real call without `Idempotency-Key`. |
| `idempotency_key_reused` | 422 | The key was used for a different call. |
| `grant_revoked` | 401 | A queued job found the OAuth grant revoked. |
| `operation_expired` | 422 | A queued job did not start within 120 seconds. |

## Live notifications

When something a named query shows changes, the brain says so on its private channel `private-mortel.brain.{brain}`, as the event `BrainChanged`. With Laravel Echo that is `Echo.private('mortel.brain.' + brain).listen('.BrainChanged', ...)`.

```json
{"brain": "01j...", "queries": ["entities"]}
```

That is the whole payload: query names only, with no values and no item refs, because every member of the brain hears it. The client reads those queries again under its own rights; with the ETag an unchanged answer costs a 304. `"queries": ["*"]` means: read everything again. It is sent after every policy or role change and once at the end of a projector replay. The message goes out after the database transaction commits, so a rolled-back change sends nothing.

`MortelApi::channels()` lets a user listen only when the channel's brain is the current tenant, the user is a member of it, and an OAuth grant, when there is one, is still active.

The default `/broadcasting/auth` route runs in the `web` group, so it only knows session users. For bearer clients, add `/api/broadcasting/auth` behind the API auth and tenancy middleware, for example in `bootstrap/app.php`:

```php
->withBroadcasting(__DIR__.'/../routes/channels.php', [
    'prefix' => 'api',
    'middleware' => ['auth:api', InitializeTenancyByRequestData::class],
])
```

That route authorises every channel in `routes/channels.php`, not only the Mortel channel. Restrict it to `private-mortel.brain.*` channels, for example with a middleware that checks the `channel_name` field of the request. The framework does not ship that check.

## MCP

`Mortel\MCP\Servers\MortelServer` serves the same queries and actions as MCP tools. Mount it in `routes/ai.php`, after the host's auth and tenancy middleware and with `ResolveApiContext`:

```php
use Laravel\Mcp\Facades\Mcp;
use Mortel\Api\Http\Middleware\ResolveApiContext;
use Mortel\MCP\Servers\MortelServer;

Mcp::web('/mcp/mortel', MortelServer::class)->middleware([
    'auth:api',
    InitializeTenancyByRequestData::class,
    ResolveApiContext::class,
]);
```

A query becomes `query_<name>` and an action becomes `action_<name>`, with every dot replaced by a dash: `entities` is `query_entities` and `entities.rename` is `action_entities-rename`. A tool the user may not use is not listed and cannot be called. Query tools take `page` and `per_page` and return the HTTP query body, without ETags. Action tools take `idempotency_key`, `input` and, for an action with an item type, `item` and `expected_revision`. They return the same receipt as HTTP and share its idempotency scope, so a call can be retried over either transport without running twice. MCP has no Precognition. A refusal is an MCP tool error whose text is the HTTP error body.

## Adding your own queries and actions

A host or package adds a named query (`Mortel\Api\Queries\NamedQuery`) or a public action (`Mortel\Api\Actions\PublicAction`) in a service provider's `register()`. Both registries are bound scoped, so use `extend()`; an entry added to a resolved instance is lost in the next scope:

```php
use Mortel\Api\Actions\PublicActionRegistry;
use Mortel\Api\Queries\NamedQueryRegistry;

$this->app->extend(NamedQueryRegistry::class, fn (NamedQueryRegistry $registry, $app) => $registry->register($app->make(InvoicesQuery::class)));
$this->app->extend(PublicActionRegistry::class, fn (PublicActionRegistry $registry, $app) => $registry->register($app->make(SendInvoice::class)));
```

A registry refuses an invalid name and a second entry with a name it already has. The new entries show up in discovery, in the HTTP routes and as MCP tools. A query lists its `fields()` and the stored event classes it `watches()`; those events drive `changed.queries` and live notifications. An action's `schema()` is its only input validation, and `allowsNow()` is its state check. The full contract is in `docs/api-v1.md` in the framework repository.

## What the API refuses

- A request without a user, or from a user who is not a member of the brain.
- A guessed id. Another owner's private entity, an entity of another brain, an entity whose type the role may not read and a ref that does not exist all get the same 404.
- Fields above the user's trust level, in rows, discovery and MCP.
- A stale revision. Of two calls with the same expected revision, exactly one is `done`.
- A queued action whose right, membership or grant was revoked before the job ran.
- A second run of the same call with the same idempotency key.

## Known limits

- Lists use the row scope without the per-entity policy step of a single read. An entity visible only through such a policy is missing from lists and totals, but a call by its ref can still find it.
- Channel authorisation happens when the socket subscribes. After a revocation an open socket keeps getting these data-free messages until it disconnects. Every API read fails from that moment.
- The grant check is per OAuth client, user and brain, not per token. A queued action from a connection that was just revoked still runs when the same client has another active connection for that user and brain, within the 120-second admission window.
