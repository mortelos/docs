---
title: "MortelOS Sidecar"
nav_title: "Sidecar"
slug: "package-sidecar"
version: "0"
description: "Hands heavy chat questions to Claude Code on the asking user's own Claude account, in a closed sandbox."
section: "packages"
order: 64
status: "mvp"
audience: "developers"
package: "mortelos/sidecar"
canonical_path: "/docs/0/package-sidecar"
last_verified: "2026-10-16"
public: true
---

# MortelOS Sidecar

`mortelos/sidecar` hands heavy chat questions to Claude Code. It runs on the asking user's own Claude account, in a closed sandbox. Chat runs have a 30-minute limit by default; standalone tasks default to 600 seconds, and workflow tasks allow at most 600 seconds. The sidecar reads data only through read-only agent tools under the asker's policy and calculates in Python. Its answer and files go back into the conversation. The same runner also runs standalone tasks and `sidecar.task` workflow steps. This page describes version 0.2.6.

The user-facing texts are in Dutch. They can be changed in config and per tenant.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Handover | The `sidecar.start` agent tool. A chat run that hits its runtime or tool budget is also handed over. |
| Account connection | Each user connects their own Claude account through Claude's own sign-in flow. |
| Sandbox | Runs Claude Code with its Bash sandbox (bubblewrap and socat). The sandbox has no network domains, and web fetch and web search are off. |
| Data access | Only allow-listed agent tools with read risk that the asker's policy allows and that need no approval. Results are written as CSV files. |
| Output | Files the sidecar writes to `output/` become downloads under the answer, after hardening and checks. |
| Follow-up | When the sidecar asks a question, the user's reply goes to a new sidecar run. |
| Tasks | Standalone tasks without a conversation, and the `sidecar.task` workflow step. |
| Settings | An admin settings page and per-tenant overrides in `tenant_settings`. |

## Requirements

Composer: PHP 8.4 with `ext-dom`, `mortelos/framework` `^0.6.31`, `mortelos/chat` `~0.3.21.5 || ^0.3.27 || ^0.4 || ^0.5`, `mortelos/ui` `^0.4.6 || ^0.5` and Livewire `^4.0`. The framework constraint stays within 0.6; it does not include 0.7.0.

Server:

- **Claude Code.** Install the Claude Code CLI and set its absolute path in `SIDECAR_CLAUDE_BINARY`. PHP-FPM gives web requests no `PATH`, so a bare name is not found when a user connects.
- **Sandbox tools.** `bwrap` and `socat` must be on the `PATH` that Claude gets, or the run does not start. Use `SIDECAR_SANDBOX_PATH_PREPEND` or `SIDECAR_BWRAP_DIR` when the system has none.
- **Python 3.** Set `SIDECAR_PYTHON`. The default instructions use matplotlib, openpyxl, python-docx and reportlab for output files, and skip a format when a library is missing. If a Python environment lives under a denied path such as `/home`, add it with `SIDECAR_SANDBOX_READ_PATHS`.
- **Queue worker.** Run a worker for the sidecar queue (`SIDECAR_QUEUE`, default `sidecar`). The job timeout is `max_seconds` plus 60. Set the queue connection's `retry_after` clearly above that, for example 1920 for the default 1800, or a second worker picks the job up again.
- **Shared cache.** The run slots are cache locks in the default cache store. All workers must share it (redis or database, not array or file per host).
- **Event sourcing.** A user's Claude connection is a personal channel of type `sidecar_personal`. List the framework's `ChannelProjector` in `config/event-sourcing.php`.
- **Tenant settings.** Per-tenant overrides need the `tenant_settings` table. Without it, the package reads config only.

## Install

```bash
composer require mortelos/sidecar
php artisan vendor:publish --tag=sidecar-config
php artisan migrate
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The service provider is auto-discovered and loads the package migrations. In a published `config/sidecar.php`, the `texts` block is merged per key, so you can keep only the texts you change. Every other block replaces the package block as a whole, such as a shorter `sandbox_deny_read`.

## Switch It On

1. Set `SIDECAR_ENABLED=true`.
2. Set `pilot_roles` to the role names that may use the sidecar in the tenant. An empty list allows nobody.
3. Allow `agent_tool:sidecar.start` in the agent tool policy for those roles. It is deny-by-default.
4. Set `tool_allowlist` to `fnmatch()` patterns of the read tools the sidecar may call. An empty list gives it no data tools.
5. Let users connect their Claude account.

## Connecting Claude

The package registers two Livewire pages under `route_prefix` (default `deep-research`) with `route_middleware` (default `web`, `auth`):

| Path | Route name | Purpose |
| --- | --- | --- |
| `/koppelen` | `sidecar.connect` | The step where a user connects their Claude account. |
| `/instellingen` | `sidecar.settings` | The admin settings page. |

The Livewire components are `sidecar::connect-account`, `sidecar::account-card`, `sidecar::settings` and `sidecar::pages.connect-account`. The account card shows connection state and quota, with reconnect and disconnect buttons.

To prompt users after login, call `Mortelos\Sidecar\Connection\SidecarOnboarding::shouldPrompt($userId)` and redirect to `sidecar.connect` when it returns `true`. It is true when the sidecar is on, the user has a pilot role, the account is not connected and the user did not skip the step in this session.

The `sidecar.connect` gate decides who may connect. By default it allows pilot users. The package defines it only when the host has not, so a host can define its own, for example to refuse impersonation sessions.

| Setting | Effect |
| --- | --- |
| `login.allowed_subscription_types` | Claude plans that may connect: `pro`, `max`, `team`, `enterprise`. Empty allows every plan. |
| `login.allowed_email_domains` | E-mail domains of accounts that may connect. Empty allows every account. Users with the `sidecar.connect-external` ability may connect any account. |
| `login.code_ttl_seconds` | How long a started login waits for the pasted code. Default 900. |

The login lives in a per-user directory under `login.login_root` (`SIDECAR_LOGIN_ROOT`). Only the Claude binary reads and refreshes it. The app never reads, copies or stores the credentials; the database holds connection state only. Keep `login_root` outside anything the sandbox may read. The runner always denies the sandbox access to the work root and the login directories.

Users on a personal Claude plan (Free, Pro or Max) see a notice to switch off model training in their Claude privacy settings.

## How A Question Reaches The Sidecar

A chat run may go to the sidecar when the sidecar is on and the run is a chat run with a conversation. The asker must have a pilot role, a working Claude connection, Claude quota and no other sidecar queued or running.

- **Tool.** The chat model calls `sidecar.start` with a `reason` and a list of `assumptions`. The chat ends its turn with a short handoff text. The sidecar is queued once the chat run has completed and posts its answer in the conversation.
- **Safety net.** A chat run that hits its runtime or tool-iteration budget goes to the sidecar when the asker may use it. Otherwise it ends with a message that says why, and what was found so far.
- **Not connected.** The question is parked (only the latest one) and the chat shows a connect widget. Once the user connects, the question continues by itself.
- **Busy.** Each user has at most one sidecar at a time. The chat tells the user that an earlier question is still being worked on.
- **Follow-up.** When the sidecar asks the user a question, the reply goes to a new sidecar run. The answer is added to the assumptions.

The sidecar gets the original message, the conversation, the chat's earlier steps and the assumptions in `handoff/`. Tool results go to `results/` as CSV. Claude receives the `instructions` setting as an addition to its system prompt and the `task_prompt` setting as its prompt.

## Output Files

Files the sidecar writes directly in `output/` become downloads under its answer. Other files are ignored.

- Allowed extensions: `csv`, `xlsx`, `docx`, `pdf`, `html`, `svg`, `png`, `jpg`, `md`, `txt`. `output.extensions` can only narrow this list.
- At most 10 files (`output.max_files`) of up to 10 MB each (`output.max_file_bytes`).
- An HTML file is rebuilt as a static page. Scripts, forms and everything that points outside the page are removed. A standalone SVG is rebuilt as a static drawing in the same way.
- A Word, Excel, PDF or Markdown file with a link or reference to outside is not handed out.
- A CSV is attached unchanged unless it has cells that look like formulas. Those cells are prefixed with an apostrophe. An outside reference alone does not cause refusal. A read or processing failure does.

## Limits

| Limit | Default |
| --- | --- |
| Run time (`max_seconds`) | 1800 seconds, at least 60. Claude is interrupted 60 seconds before the deadline (`interrupt_lead_seconds`). |
| Runs at the same time on the server (`max_concurrent`) | 2 in total, at most 1 per user. |
| Wait for a free slot (`max_queue_seconds`) | 3600 seconds. After that, the user is asked to try again later. |
| Network | None for Bash. Web fetch and web search are denied. |
| Engine (`engine`) | `claude` only. An unknown engine fails when the sidecar is used. |
| Quota | The user's own Claude quota. When it runs out, the user cannot use the sidecar until the reset time. |

## Settings Per Tenant

A tenant setting `sidecar.<key>` overrides `config('sidecar.<key>')`, so limits, roles, instructions and texts change without a deploy. Jobs read tenant values only inside tenant context.

Keys that decide what the sandbox can reach come only from config: `engine`, `claude_binary`, `bwrap_dir`, `sandbox_path_prepend`, `sandbox_deny_read`, `sandbox_read_paths`, `python_binary`, `work_root` and `login.login_root`.

The settings page edits these keys per tenant:

| Group | Keys |
| --- | --- |
| Access | `enabled`, `pilot_roles`, `login.allowed_subscription_types`, `login.allowed_email_domains` |
| Limits | `max_seconds`, `max_concurrent` |
| Model and instructions | `model`, `effort`, `instructions`, `start_tool_guidance` |
| Data | `tool_allowlist` |
| Tool sharing | `skills.promotion_min_uses`, `skills.shared_cap`, `skills.expiry_days` |
| Texts | The user-facing texts |

Saving goes through the framework's tenant setting writer, which requires `tenant-settings.write`. Only values that differ from the current value are written.

The name the sidecar uses in texts comes from `assistant_name` (`SIDECAR_ASSISTANT_NAME`), then `mortel.assistant_name`, then `app.name`.

## Configuration

| Key | Environment variable | Default |
| --- | --- | --- |
| `enabled` | `SIDECAR_ENABLED` | `false` |
| `pilot_roles` | | `[]` |
| `queue` | `SIDECAR_QUEUE` | `sidecar` |
| `max_seconds`, `max_concurrent`, `max_queue_seconds` | | `1800`, `2`, `3600` |
| `engine` | `SIDECAR_ENGINE` | `claude` |
| `claude_binary` | `SIDECAR_CLAUDE_BINARY` | `claude` |
| `model`, `effort` | | `opus`, `high` |
| `sandbox_read_paths` | `SIDECAR_SANDBOX_READ_PATHS` (colon-separated, added) | `/usr` |
| `sandbox_deny_read` | | `/home`, `/Users`, `/root`, `/run`, `/var/run`, `/var/www`, `/srv` |
| `sandbox_path_prepend` | `SIDECAR_SANDBOX_PATH_PREPEND` | empty |
| `bwrap_dir` | `SIDECAR_BWRAP_DIR` | empty |
| `python_binary` | `SIDECAR_PYTHON` | `python3` |
| `work_root` | `SIDECAR_WORK_ROOT` | `storage/app/sidecar` |
| `display_timezone` | `SIDECAR_DISPLAY_TIMEZONE` | `Europe/Amsterdam` |
| `route_prefix`, `route_middleware` | | `deep-research`, `['web', 'auth']` |
| `tool_allowlist` | | `[]` |
| `memory.enabled` | `SIDECAR_MEMORY_ENABLED` | `false` |
| `output.extensions`, `output.max_files`, `output.max_file_bytes` | | see Output Files |
| `tasks.*` | | see Standalone Tasks |
| `login.login_root` | `SIDECAR_LOGIN_ROOT` | `storage/app/sidecar-logins` |

The sandbox may read everything that is not in `sandbox_deny_read`. Deny every directory that holds unix sockets, because the sandbox must allow unix sockets. `sandbox_read_paths` re-allows paths inside a denied area, and none of them may contain a denied path.

## Extension Points

| Seam | Use it to |
| --- | --- |
| `Mortelos\Sidecar\Contracts\SidecarMemory` | Supply the shared definitions the sidecar starts from, in `memory/`, and receive proposed changes for review. Nothing is applied directly. The default is no memory; bind your own and set `memory.enabled`. |
| `Mortelos\Sidecar\Contracts\SidecarRunner` | Change what happens around a run by decorating the binding with `app()->extend(SidecarRunner::class, ...)`. Do not subclass the engine runner. Write into the workdir only through `Mortelos\Sidecar\Runner\WorkdirFiles`. |
| `Mortelos\Sidecar\Tasks\TaskPreparation` | Stage task input and validate task results. The default stages authorized artifacts. |
| `Mortelos\Sidecar\Tasks\TaskDelivery` | Deliver a finished task. The default makes an inbox card or emits an event. |
| `sidecar.connect` gate | Decide who may connect a Claude account. |
| `Mortelos\Sidecar\Tasks\TaskResultReady` event | React when a task has a durable result. |

## Standalone Tasks

A host can run the sidecar without a conversation:

```php
use Mortelos\Sidecar\Tasks\StartTask;
use Mortelos\Sidecar\Tasks\TaskRequest;

$task = app(StartTask::class)->start(new TaskRequest(
    tenantId: $tenantId,
    userId: $userId,
    identity: 'weekly-summary:2026-42',
    instructions: 'Answer only from the supplied files.',
    prompt: 'Summarise the input files.',
    tools: [],
    destination: 'result',
    jsonResult: true,
));
```

- The task runs as a real user. That user must exist, have a role in the tenant and have pilot access, and the sidecar must be on. There is no system actor.
- `identity` is unique per tenant, up to 191 characters. Starting the same task again returns the same task and dispatches its job again. A different definition or user under the same identity fails. A new paid attempt needs a new identity.
- `model` must be empty or equal the installation's model. The installation's model and effort are saved with the task. `deadlineSeconds` defaults to 600 and is capped by `max_seconds`.
- `tools` holds exact registry names. An empty list gives the task no tools.
- `inputs` holds `{artifact_id, checksum, path}` references with a SHA-256 checksum. Inputs and outputs must be ordinary text files within the `tasks.*` limits. Links, hidden configuration, credential file names and binary files are refused.
- Personal memory and personal skills are off unless `personalMemory` or `personalSkills` is set. Memory also needs `memory.enabled`.

| `tasks.*` key | Default |
| --- | --- |
| `input_max_files` | 1000 |
| `input_max_bytes` | 20 MB |
| `output_max_files` | 100 |
| `output_max_bytes` | 10 MB |
| `preview_max_chars` | 4000 |

The outcome is one of `answered`, `question`, `usage_limit_reached`, `login_required`, `deadline_exceeded`, `stopped`, `failed` or `invalid_output`. Only `answered` is a success. The answer and output files are stored as an internally classified JSON artifact owned by the user, before delivery.

Read the result with `app(TaskResults::class)->read($task, $userId)`, inside the task's tenant and with the real executing user. Do not expose it as a public endpoint.

| `destination` | Delivery |
| --- | --- |
| `inbox` (default) | One internal inbox card for the user, with a preview of the answer. |
| `result` | Only the `TaskResultReady(tenantId, taskId, agentRunId)` event. Listeners read the durable result. |

A failed delivery leaves `delivery_error` set to `delivery_failed` and keeps the result. Dispatch `DeliverTaskJob($tenantId, $taskId)` to retry the delivery only. A custom `TaskDelivery` must be idempotent per task and do local work only.

## Workflow Step

The package registers the `sidecar.task` step type with the framework's workflow step registry. The host must first:

- Run the framework's workflow-step wait migration.
- Register synchronous `WorkflowRunProjector` and `WorkflowStepWaitProjector` projectors with Spatie event sourcing, and include `WorkflowStepWaitRegistered` and `WorkflowStepWaitResolved` in `event-sourcing.event_class_map`.
- Bind `TenantContext` and `TenantResolver`, and keep `mortel:workflows:run-due` running for recovery.

Example step:

```json
{
  "id": "research",
  "type": "sidecar.task",
  "config": {
    "instructions": "Research only the supplied input and return JSON.",
    "prompt": "Prepare the summary from the input files.",
    "tools": [],
    "inputs": [],
    "model": "opus",
    "effort": "high",
    "deadlineSeconds": 600,
    "jsonResult": true
  },
  "on_failure": {"behavior": "abort"}
}
```

- `model` and `effort` are required and must equal the installation's settings when the task is created.
- `deadlineSeconds` must be from 1 to 600, and is capped by the installation limit.
- The workflow run snapshot must contain `executor_user_id`. The host must bind the framework's `WorkflowStepAuthorizer`.
- The step reads its config from the run's saved definition, not from the live workflow or the payload.
- The task identity is `workflow:<run-id>:<step-index>`. A workflow retry reuses the task and never starts another paid run.
- Workflow tasks always use `destination` `result`, with personal memory and personal skills off. The `tools` list still grants the named agent tools; an empty list grants none.

Only `answered` completes the step, with this result:

```json
{"task_id": "<task-id>", "artifact_id": "<artifact-id>", "checksum": "<sha256>"}
```

Every other outcome fails the step as `task_<outcome>` and follows the step's failure behaviour. A later step reads the bytes with `TaskResults::read()` as the current executor. Keep the framework's workflow recovery running: it completes a step whose notification was missed.

## Upgrading to 0.3.0

- The settings route prefix changes from `diepteonderzoek` to `deep-research`. Update bookmarks and reverse proxy rules that use the old path.
- The query parameter of the connect return URL changes from `gesprek` to `conversation`. Update links that you build by hand.
- Inbox items for completed sidecar tasks use the type `notification` instead of `notificatie`. Update host code that filters on this type.
- Require `mortelos/framework:^0.7` alongside this package. If the host publishes `config/mortel.php`, rename `inbox.max_per_bron_per_dag` to `inbox.max_per_source_per_day`. Follow the framework 0.7 migration steps; see [Upgrading to 0.7.0](/docs/0/package-framework#upgrading-to-070).
- Upgrade `mortelos/chat` to `^0.6` together with this package. Chat 0.5 requires framework 0.6 and cannot be installed with framework 0.7.

## Outlets

An outlet is a place where you extend a package from your app without changing it; read [Outlets](/docs/0/outlets) for how each kind works. List the outlets the installed version declares with:

```bash
php artisan mortelos:outlets:list sidecar
```

## Boundaries

The package owns the handover, the Claude connection flow, the sandboxed runner, output checks, settings, standalone tasks and the workflow step. The host app owns the server setup (Claude Code, bubblewrap, socat, Python, queue worker, shared cache), pilot roles and agent tool policies, which tools are allow-listed, the memory source, task preparation and validation, and which users may run tasks. The sidecar never changes data: it reads through read-only tools and proposes memory changes for review.
