---
title: "MortelOS Framework"
nav_title: "Framework"
slug: "package-framework"
version: "0"
description: "Core MortelOS package for entities, links, events, projections, tenant primitives and runtime contracts."
section: "packages"
order: 52
status: "mvp"
audience: "developers"
package: "mortelos/framework"
canonical_path: "/docs/0/package-framework"
last_verified: "2026-10-15"
public: true
---

# MortelOS Framework

`mortelos/framework` is the core library for MortelOS primitives and runtime behavior.

The current PHP namespace can still be `Mortel\...` during the external package naming transition. Use `mortelos/framework` as the public package name in docs, plans and package decisions.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Entities and links | Entity registry, typed entity relationships and shared entity behavior. |
| Events and projections | Event-sourced facts, projection plumbing and reusable application services. |
| Tenant primitives | Tenant-aware context, access resolution and installation-level runtime contracts. |
| Agent runtime | MCP runtime, Laravel AI integration and safe workspace tool surfaces. |
| Document services | Shared framework services such as PDF, markdown and diff support where those belong below a feature package. |

## Install

```bash
composer require mortelos/framework
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Outlets

An outlet is a place where you extend `mortelos/framework` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.6.41; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list framework
```

| Kind | Name | What it is for |
| --- | --- | --- |
| contract | `Mortel\Agent\Model\AgentModelGateway` | Asks the model for the next turn of an agent run. Decorate it with `extend()` for a veto, a tool filter or a retry. Default: `DefaultAgentModelGateway`. |
| contract | `Mortel\Agent\AgentLoopRunner` | Runs an agent run and delivers a final message. Decorate it with `extend()` for work before or after. A binding on `Mortel\Agent\AgentLoop` keeps working. Default: `AgentLoop`. |
| contract | `Mortel\Agent\ContextPackBuilderContract` | Builds the context pack of a run. Decorate it with `extend()` to add or hide items or to replace the pack. A binding on `Mortel\Agent\ContextPackBuilder` keeps working. Default: `ContextPackBuilder`. |
| contract | `Mortel\Actions\Inbox\AuthorizeInboxItemAccess` | Decides who may see and handle an inbox item. Bind your own subclass, for example `OwnerScopedInboxItemAccess`. Without a binding everything is denied. Default: `DenyAllInboxItemAccess`. |
| contract | `Mortel\Actions\Inbox\InboxRejectionFollowUp` | Decides whether a rejected inbox item gets a revised proposal and a learned pattern. Not bound: every rejection gets that follow-up. |
| contract | `Mortel\Actions\Memory\ChatMemoryProposalGate` | Decides whether a chat run may propose memories. Not bound: every chat run may. |
| contract | `Mortel\Support\Artifacts\ArtifactDiskResolver` | Picks the storage backend for a new artifact; the disk follows the profile in `mortel.artifacts.profiles`. Not bound: the caller's choice or the default backend. |
| contract | `Mortel\Contracts\OAuthCredentialsResolver` | Gives the OAuth app details (client ID, secret, redirect) of a provider; by default from `services.<provider>.*`. Bind your own for details per tenant or per user. Default: `ConfigOAuthCredentialsResolver`. |
| contract | `Mortel\Agent\Streaming\FinalAnswerStreamer` | Streams the final answer of an agent run to the chat. Default: `LaravelAiFinalAnswerStreamer`. |
| contract | `Mortel\Agent\TurnGuards\AgentTurnGuardContract` | Checks every model turn before a tool runs. Default: `DefaultAgentTurnGuard`. |
| contract | `Mortel\Agent\Escalation\AgentRunEscalation` | Takes over a run that has used up its budget. Default: `NullAgentRunEscalation`. |
| contract | `Mortel\Agent\Escalation\AgentRunTakeover` | Takes over a run before the model is asked. Default: `NullAgentRunTakeover`. |
| contract | `Mortel\Contracts\TenantResolver` | Gives the current tenant and its details. Default: `NullTenantResolver`. |
| contract | `Mortel\Contracts\TenantContext` | Runs work inside the right tenant; by default based on `mortel.models.tenant`. Default: `SingleTenantContext`. |
| contract | `Mortel\Contracts\SendEmailInboxItemDispatcher` | Sends the mail behind an approved `send_email` inbox item. Default: `ConfiguredSendEmailInboxItemDispatcher`. |
| contract | `Mortel\Actions\Memory\MemoryCandidateFetcher` | Finds candidate memories for recall. Default: `PgvectorMemoryCandidateFetcher`. |
| contract | `Mortel\Contracts\Workflows\WorkflowStepAuthorizer` | Gives workflow steps permission to run. Not bound: every step that asks is denied. |
| contract | `Mortel\Contracts\TenantInvitationService` | Invites a user into a tenant. No default: the host must bind it. |
| registry | `Mortel\Agent\Tools\AgentToolRegistry` | Add an agent tool (`AgentToolContract`, optionally with a capability, effect or preflight contract) through `extend()`. Register with `register()`. |
| registry | `Mortel\Agent\Capabilities\AgentCapabilityRegistry` | Add a capability definition through `extend()`. Register with `register()`. |
| registry | `Mortel\Actions\Workflows\WorkflowStepHandlerRegistry` | Add a workflow step handler; extend `AbstractWorkflowStepHandler` for the helpers. Register with `register()`. |
| registry | `Mortel\Support\NavigationItemRegistry` | Add a `NavigationItemProvider` to a menu section. Register with `register()`. |
| registry | `Mortel\Contracts\ChannelDriverRegistry` | Add a channel driver (`ChannelDriver`). Register with `register()`. |
| registry | `Mortel\Contracts\ConnectorDataRequestProviderRegistry` | Add a provider for external data requests. Register with `register()`. |
| registry | `Mortel\Contracts\ConnectorSetupProviderRegistry` | Add a connector setup provider; when it also implements `ConnectorSetupHelpProvider`, the setup shows its help text. Register with `register()`. |
| registry | `Mortel\AppPlatform\Support\OsAppRegistry` | Add an OS app (`OsApp`). Register with `register()`. |
| registry | `Mortel\Agent\Tools\AgentToolRule` | A rule that decides per tool before the policy: deny, allow or no opinion (`null`). Careful: a rule that allows overrules the policy. Tag your class with `$app->tag([YourClass::class], AgentToolRule::class)`. |
| registry | `Mortel\AI\Agents\AgentInstructionsContributor` | Adds text per run to the system instruction of the agent runtime. Tag your class with `$app->tag([YourClass::class], AgentInstructionsContributor::class)`. |
| registry | `Mortel\Support\Artifacts\ArtifactAccessRule` | An extra deny rule on top of an artifact's owner, grant and policy. Tag your class with `$app->tag([YourClass::class], ArtifactAccessRule::class)`. |
| config | `mortel` | All keys in `config/mortel.php`, including `mortel.pdf.view` (the view of a PDF export), `mortel.assistant_name`, `mortel.agent.tool_aliases` and `mortel.models`. The old tag `os-core-config` still works. Publish with `php artisan vendor:publish --tag=mortel-config`. |
| event | `Mortel\Events\AgentRunQueuing` | Just before an agent run is saved. A listener may change the attributes, for example to pin the scope in the context snapshot. |
| event | `Mortel\Events\AgentRunStarting` | Inside the lock, just before a run goes from queued to running. An exception keeps the run queued. |
| event | `Mortel\Events\AgentRunFinished` | Once when a run ends (completed, failed or cancelled), after commit. Carries the run and the outcome. |
| event | `Mortel\Events\AgentRunStreamedEvent` | A piece of the final answer of an agent run was streamed. Broadcast to the browser. |
| event | `Mortel\Events\AgentRunUpdatedEvent` | An agent run or one of its steps changed. Broadcast to the browser. |
| event | `Mortel\Events\AiAction\AiActionAccepted` | An AI proposal was accepted. Stored fact (event sourcing). |
| event | `Mortel\Events\AiAction\AiActionProposed` | The AI proposed an action. Stored fact (event sourcing). |
| event | `Mortel\Events\AiAction\AiActionRejected` | An AI proposal was rejected. Stored fact (event sourcing). |
| event | `Mortel\Events\Channel\ChannelCreated` | A channel was created. Stored fact (event sourcing). |
| event | `Mortel\Events\Channel\ChannelDeleted` | A channel was deleted. Stored fact (event sourcing). |
| event | `Mortel\Events\Channel\ChannelUpdated` | A channel was changed. Stored fact (event sourcing). |
| event | `Mortel\Events\Channel\MailReceived` | A mail came in on a channel. Stored fact (event sourcing). |
| event | `Mortel\Events\Channel\MailSent` | A mail was sent through a channel. Stored fact (event sourcing). |
| event | `Mortel\Events\DashboardRefreshEvent` | The dashboard should reload. Broadcast to the browser. |
| event | `Mortel\Events\Document\DocumentContentUpdated` | The content of a document changed. Stored fact (event sourcing). |
| event | `Mortel\Events\Document\DocumentCreated` | A document was created. Stored fact (event sourcing). |
| event | `Mortel\Events\Document\DocumentSignedOff` | A document was signed off. Stored fact (event sourcing). |
| event | `Mortel\Events\Entity\EntityCreated` | An entity was created. Stored fact (event sourcing). |
| event | `Mortel\Events\Entity\EntityExported` | Entities were exported. Stored fact (event sourcing). |
| event | `Mortel\Events\Entity\EntityStatusChanged` | The status of an entity changed. Stored fact (event sourcing). |
| event | `Mortel\Events\Entity\EntityUpdated` | An entity was changed. Stored fact (event sourcing). |
| event | `Mortel\Events\Experiment\ExperimentCompleted` | An experiment was completed. Stored fact (event sourcing). |
| event | `Mortel\Events\Experiment\ExperimentStarted` | An experiment was started. Stored fact (event sourcing). |
| event | `Mortel\Events\ExternalActionCompleted` | An external action succeeded. Broadcast to the browser. |
| event | `Mortel\Events\ExternalActionFailed` | An external action failed. Broadcast to the browser. |
| event | `Mortel\Events\Inbox\InboxAttentionChanged` | The attention status of an inbox item changed. Stored fact (event sourcing). |
| event | `Mortel\Events\Inbox\InboxBronAfgeremd` | A source reached the daily limit of the inbox gate; after commit. |
| event | `Mortel\Events\Inbox\InboxFindingPublished` | A finding was published in the inbox. Stored fact (event sourcing). |
| event | `Mortel\Events\Inbox\InboxFollowUpRecorded` | A follow-up was recorded on an inbox item. Stored fact (event sourcing). |
| event | `Mortel\Events\Inbox\InboxHumanResponseRecorded` | A human response was recorded on an inbox item. Stored fact (event sourcing). |
| event | `Mortel\Events\Inbox\InboxItemBlocked` | An inbox item was blocked. Stored fact (event sourcing). |
| event | `Mortel\Events\Inbox\InboxItemReclassified` | An inbox item was classified again. Stored fact (event sourcing). |
| event | `Mortel\Events\InboxItemApproved` | An inbox item was approved. Broadcast to the browser. |
| event | `Mortel\Events\InboxItemCreated` | An inbox item was created. Broadcast to the browser. |
| event | `Mortel\Events\InboxItemRejected` | An inbox item was rejected. Broadcast to the browser. |
| event | `Mortel\Events\Link\LinkCreated` | A link between entities was made. Stored fact (event sourcing). |
| event | `Mortel\Events\Link\LinkRemoved` | A link between entities was removed. Stored fact (event sourcing). |
| event | `Mortel\Events\Metric\MetricRecorded` | A metric value was recorded. Stored fact (event sourcing). |
| event | `Mortel\Events\Policy\PolicyCreated` | A policy was created. Stored fact (event sourcing). |
| event | `Mortel\Events\Policy\PolicyDeleted` | A policy was deleted. Stored fact (event sourcing). |
| event | `Mortel\Events\Policy\PolicyUpdated` | A policy was changed. Stored fact (event sourcing). |
| event | `Mortel\Events\QuestionAnsweredEvent` | A question was answered. Broadcast to the browser. |
| event | `Mortel\Events\QuestionAnsweringFailedEvent` | Answering a question failed. Broadcast to the browser. |
| event | `Mortel\Events\Role\RoleCreated` | A role was created. Stored fact (event sourcing). |
| event | `Mortel\Events\Role\RoleDeleted` | A role was deleted. Stored fact (event sourcing). |
| event | `Mortel\Events\Role\RoleUpdated` | A role was changed. Stored fact (event sourcing). |
| event | `Mortel\Events\Transcript\TranscriptImported` | A transcript was imported. Stored fact (event sourcing). |
| event | `Mortel\Events\Transcript\TranscriptPurged` | A transcript was erased. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowCreated` | A workflow was created. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunCompleted` | A workflow run was completed. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunCreated` | A workflow run was created. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunDefinitionSnapshotted` | The definition of a workflow run was recorded at its start. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunFailed` | A workflow run failed. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunPaused` | A workflow run was paused. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunResumed` | A workflow run was resumed. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunStarted` | A workflow run was started. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunStepCompleted` | A step of a workflow run was completed. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowRunStepFailed` | A step of a workflow run failed. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowScheduleClaimed` | A worker claimed a scheduled workflow run. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowScheduleConfigured` | The schedule of a workflow was set. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowStepWaitRegistered` | A workflow step waits for something from outside. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowStepWaitResolved` | What a workflow step was waiting for has arrived. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowTriggered` | A workflow was triggered. Stored fact (event sourcing). |
| event | `Mortel\Events\Workflow\WorkflowUpdated` | A workflow was changed. Stored fact (event sourcing). |

Writing a tenant setting checks the gate ability `tenant-settings.write`; define it in the host. A gate is not an outlet in the manifest.

## Boundaries

Use this package for OS-level primitives that multiple MortelOS installations can share. Keep customer-specific policy defaults, branding, local orchestration and one-off integrations in the host app.

## Action results, undo and actors

Since v0.6.38 every write goes through the same few types. The starter shows them in use: `app/Actions/Invitations/InviteUser.php` and `app/Providers/StarterServiceProvider.php`.

- `Mortel\Actions\ActionResult` is what an Action returns. Its status is `done`, `approval_needed`, `failed` or `skipped`. A refusal by policy stays an exception.
- `Mortel\Actions\Undo` is the recipe to reverse a change: a counter-Action class, its params and a `validUntil`. It lives in the result, not in the event.
- `Mortel\Actions\UndoRegistry` maps a counter-Action to its handler. The counter-Action records its own event, and that event is the audit.
- `Mortel\Support\Actor` says who acts, in four forms: `Actor::user()` (the bare user id), `Actor::agent()` (`agent:<run-id>`), `Actor::system()` (`system:<process>`) and `Actor::bridge()` (`bridge:<name>`).
- `EventContext::within($actor, $reason, $work)` fills actor, reason, org and branch around the work and restores the previous context afterwards.
- `PolicyScope::Action` is the policy scope of host Action abilities such as `invitations.manage`.
- `config('mortel.events.schema_namespaces')` opts a host namespace in to payload validation. The schema file sits next to the event class, and its name follows the class name: `InvitationCreated` is validated against `invitation.created.v1.schema.json`.
- `handle()` is the method on every Action. `execute()` still works on the Actions that had it.
- The `events.actor` column is 255 characters wide from v0.6.41, so `agent:`, `system:` and `bridge:` actors fit.

## Future Optimization Notes

When MortelOS optimization work moves from application-level cleanup to database-level tuning, consider PostgreSQL functions or procedures for small, data-heavy invariants. Good candidates are planner task ordering, append-to-day task planning, entity search ranking, memory recall candidate lookup, import-registry claims, AI action statistic increments and version publication counters.

Do not move broad workflows into the database by default. Keep external API calls, AI calls, embeddings, queue dispatch, Laravel events, mail delivery, policy orchestration and UI behavior in application code. Database routines should only own logic that benefits from atomicity, indexes, locking or reduced round trips.

## Runtime mount

Host apps mount the framework MCP server when operate mode is enabled:

```php
use Laravel\Mcp\Facades\Mcp;
use Mortel\MCP\Servers\UteqOSServer;

Mcp::oauthRoutes();

Mcp::web('/mcp/mortelos', UteqOSServer::class)
    ->middleware(['auth:api']);
```

Production hosts add tenant initialization, role resolution, trust-level enforcement, data classification and throttling around that route.
