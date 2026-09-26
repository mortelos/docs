---
title: "MortelOS Policy Studio"
nav_title: "Policy Studio"
slug: "package-policy-studio"
version: "0"
description: "Proposal-first access changes from chat, with admin review and governance pages for grants, policy conditions and agent approvals."
section: "packages"
order: 61
status: "mvp"
audience: "developers"
package: "mortelos/policy-studio"
canonical_path: "/docs/0/package-policy-studio"
last_verified: "2026-09-26"
public: true
---

# MortelOS Policy Studio

`mortelos/policy-studio` turns access requests in chat into policy proposals that an admin approves or rejects. It also adds admin pages for context grants, policy conditions and agent approvals.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Chat proposals | The `policy_change_proposal` widget drafts one change for one role. The change can show or hide a sidebar navigation item, allow or deny an agent tool (optionally on a folder under a configured root), or allow or deny a chat widget. Submitting stores a pending `policy_change_proposal` inbox item and changes no policy. Submitting again from the same widget run updates that open proposal. |
| Review | Admins approve or reject pending change proposals in the widget, and change and condition proposals in the proposal queue component. Approval creates a policy through the `mortelos/framework` policy actions, or merges the actions into the existing policy for the same role, scope and resource. `PolicyProposalService::approve()` and `reject()` run the admin check themselves. Proposals carry a risk level: `low` for navigation, `high` for folder access, `medium` otherwise. |
| Governance overview | The `policy_governance_overview` widget shows roles, policy rules and navigation rules, for all roles or for one role named in the question. For each registered chat widget it checks the declared policy ability per role and flags widgets that declare none. |
| Intent detection | `AnswerPolicyChangeRequest` decides whether a chat message is a policy request or an overview question and returns the widget payload. Keyword matching runs first; unclear messages go to the `PolicyChangeIntentClassifier` contract, bound by default to the Laravel AI agent `PolicyChangeIntentAgent`. Classifier values that do not match a supplied role, navigation permission or tool action are dropped, and a classifier exception counts as no match. |
| Studio pages | Admin pages to create and revoke `view` context grants for a user, role or tenant with an optional expiry date, list agent-tool policies per role, propose condition changes on a policy, and approve or reject pending agent-run approval steps. |

## Install

```bash
composer require mortelos/policy-studio
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Chat widget | `policy_change_proposal` |
| Chat widget | `policy_governance_overview` |
| Livewire component | `policy-studio::governance.proposal-queue` |
| Route | `policy-studio.grants` (`/policy-studio/grants`) |
| Route | `policy-studio.conditions` (`/policy-studio/conditions`) |
| Route | `policy-studio.approvals` (`/policy-studio/approvals`) |
| Inbox item type | `policy_change_proposal` |
| Inbox item type | `policy_condition_proposal` |

`policy_change_proposal` uses skill `policy_change` and declares policy ability `policy.change.propose`. `policy_governance_overview` uses skill `policy_governance_overview` and declares `policy.change.view`.

The routes use the `web` and `auth` middleware. Each page shows its content only to users who pass the admin check, and scopes its queries to the active tenant. The proposal queue has no route, so embed it in a host page. In `mortelos/starter`, set `starter.governance.proposal_queue_component` to `policy-studio::governance.proposal-queue` to show it on the governance page.

## Configuration

```bash
php artisan vendor:publish --tag=policy-studio-config
```

The published file is `config/policy-studio.php`.

| Key | Purpose |
| --- | --- |
| `access_resolver` | Class with a `canManage($user, $tenantId)` method that decides who may review proposals and use the studio pages. When empty, `starter.governance.access_resolver` is used. Without either, only users with a truthy `is_super_admin` attribute pass. |
| `roots` | Folder roots that folder-scoped agent tool proposals may target, merged with the framework's `mortel.agent.os.roots`. The default root `app` points at `base_path()`. Paths must be relative to a root and cannot contain `..`. |
| `ai_intent` | `enabled`, `model`, `confidence_threshold` (default `70`) and `timeout` for the classifier. |
| `chat_widget.enabled` | Register the two chat widgets. Default `true`. |
| `studio_routes.enabled` | Register the studio routes. Default `true`. The key is not in the published file. |

## Boundaries

`mortelos/policy-studio` owns the proposal flow, the two chat widgets, the review and studio screens and the intent detection. It does not route chat messages itself: the host registers the `policy_change` skill in its chat orchestration and passes matching messages to `AnswerPolicyChangeRequest` (`shouldHandleChatMessage()`, then `ask()`). To replace the default classifier, bind your own `PolicyChangeIntentClassifier`.

`mortelos/framework` owns roles, policies, the policy actions and condition evaluator, context grants, inbox items and agent-run approvals. `mortelos/chat` owns the widget registry, widget runs and widget rendering. The host owns role names, extra navigation items, folder roots, the admin resolver and where the proposal queue and studio pages appear.

Only chat proposals and condition changes are proposal-first. The grants page writes context grants directly for admins. The condition editor validates the condition JSON and stores a `policy_condition_proposal` inbox item; approving it in the proposal queue replaces the policy's conditions through the framework's `UpdatePolicy`, and it fails without changes when the policy no longer exists. Approving either proposal type through the framework's generic inbox approval marks the item approved without changing any policy, so review them in the widget or the queue. Widget and page copy and the keyword heuristics are Dutch in this release.
