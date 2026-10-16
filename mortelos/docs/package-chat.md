---
title: "MortelOS Chat"
nav_title: "Chat"
slug: "package-chat"
version: "0"
description: "Tenant-gated chat workspace and widget runtime for MortelOS."
section: "packages"
order: 54
status: "mvp"
audience: "developers"
package: "mortelos/chat"
canonical_path: "/docs/0/package-chat"
last_verified: "2026-10-16"
public: true
---

# MortelOS Chat

`mortelos/chat` provides the tenant-gated chat workspace and the chat widget runtime.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Chat workspace | Routes, views and Livewire components for a tenant-gated workspace. |
| Widget runtime | `WidgetRegistry`, `WidgetRenderer`, widget definitions and widget run persistence. |
| Connector setup widgets | Render provider-driven connector setup forms and OAuth next actions from channel packages. |
| Widget commands | `chat:widget:make` and `chat:widget:check` for scaffolding and validation. |
| Shared storage | The `chat_widget_runs` tenant migration for widget runtime state. |

## Install

```bash
composer require mortelos/chat
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Commands

```bash
php artisan chat:widget:make demo_widget
php artisan chat:widget:check compliance_intake_start
```

## Upgrading to 0.6.0

Require `mortelos/framework:^0.7` and run the framework 0.7 upgrade migration; see [Upgrading to 0.7.0](/docs/0/package-framework#upgrading-to-070). Update chat, overviews and widget-compliance together. Update any published chat views and host integrations:

| 0.5 | 0.6 |
| --- | --- |
| `Mortel\Actions\Overzicht\SaveOverview` | `Mortel\Actions\Overview\SaveOverview` |
| browser event `overzicht-saved` | `overview-saved` |
| host listeners such as `onOverzichtSaved` | `onOverviewSaved` |
| context scopes `gesprek`, `volledig` | `conversation`, `full` (`tenant` stays) |
| model presets `snel`, `diep` | `fast`, `deep` (`medium` stays) |
| compliance visibility key `normkaart` | `rulebook` |
| host-provided Livewire component `compliance.normkaart-publication-widget` | `compliance.rulebook-publication-widget` |
| host-provided Livewire component `compliance.bronnen-toevoegen-widget` | `compliance.add-sources-widget` |

The framework migration rewrites stored context scopes and presets. Update host code that supplies these values too. Dutch customer-facing copy stays Dutch.

## Outlets

An outlet is a place where you extend `mortelos/chat` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.5.0; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list chat
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| view | `chat.thread.empty` | The empty state of a conversation, before the first message. `scrollKey` is the scroll key the host passes to `chat::thread`, so each page can show its own text. Default: an icon with a short Dutch prompt. | `conversationId`, `inboxItemId`, `scrollKey` |
| view | `chat.composer.above-input` | Above the chat text field, for chips and quick replies; on the chat page and in `chat::composer`. Default: empty. | `inputId` |
| view | `chat.composer.plus-menu` | Extra items at the bottom of the chat's plus menu (`x-mortel::menu.item`); on the chat page and in `chat::composer`. Default: empty. | `inputId` |
| view | `chat.context-panel.cards` | Extra cards below the existing cards in the context panel. A card is a `div` with a `dt` and a `dd`, because the panel is a `dl`. Default: empty. | `conversationId`, `conversableType`, `conversableId`, `agentRunId` |
| contract | `Uteq\Chat\Contracts\WidgetMessageResolver` | Decides which widget belongs to a chat message. Default: `DefaultWidgetMessageResolver`. |  |
| contract | `Uteq\Chat\Contracts\MessageFeedbackHandler` | Receives a "this is wrong" report on an answer. Set your class in `uteq-chat.feedback_handler`; without a class the report is switched off. |  |
| contract | `Uteq\Chat\Contracts\ProvidesApprovalCard` | Lets a writing agent tool write its own approval card in plain language; without it chat shows a generic card. |  |
| registry | `Uteq\Chat\Widgets\WidgetRegistry` | Add a `WidgetDefinition` or `WidgetManifest` with `extend()` in a provider, or through `uteq-chat.widgets.definitions`. Register with `register()`. |  |
| config | `uteq-chat` | All keys in `config/uteq-chat.php`. Publish with `php artisan vendor:publish --tag=uteq-chat-config`. |  |

Chat also listens to and sends Livewire browser events that a host can use: `chat-question-submitted` (send a question to the chat), `chat-question-accepted`, `chat-composer-focus`, `chat-composer-reset` and `open-conversation-panel`. The gate ability `chat.tool-calls.view` decides how much of the tool calls a user sees: when the host does not define it, everyone sees the technical view; when it does, users it allows see both views and the others see a summary. These are not outlets in the manifest, so they can change in a minor release; the changelog of the package says when.

## Boundaries

Use this package for generic chat infrastructure. Put domain-specific widgets in dedicated packages such as `mortelos/widget-compliance` or `mortelos/widget-document-feedback`.
