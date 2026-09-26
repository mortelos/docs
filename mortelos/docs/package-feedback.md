---
title: "MortelOS Feedback"
nav_title: "Feedback"
slug: "package-feedback"
version: "0"
description: "In-app feedback capture with channel-based delivery and per-tenant override for MortelOS."
section: "packages"
order: 60
status: "mvp"
audience: "developers"
package: "mortelos/feedback"
canonical_path: "/docs/0/package-feedback"
last_verified: "2026-09-26"
public: true
---

# MortelOS Feedback

`mortelos/feedback` lets signed-in users report a bug, suggestion or question from any page and delivers each report through a MortelOS channel.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Feedback widget | A floating button and form for signed-in users: type, title and optional description. |
| Page context | An optional inspector script: the user selects an element, a screenshot is taken through the browser's screen-share prompt, and recent console errors, failed `fetch` requests and form fill state (not field values) are attached. |
| Redaction | Filter passwords, tokens, secrets and similar values from URLs and context in the browser and again on the server. |
| Report storage | The `feedback_hub_reports` table, a `FH-<year>-<number>` reference per report and screenshots on a configurable disk. |
| Channel delivery | A queued job with retries sends a plain-text summary, with the screenshot attached, through the tenant's designated channel or the central default channel. |
| Admin pages | A paginated report list and a detail page with status, context and screenshot. |

## Install

```bash
composer require mortelos/feedback
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Commands

```bash
php artisan feedback-hub:install
php artisan vendor:publish --tag=mortelos-feedback-migrations
php artisan feedback:central-channel <type> <destination> --credential=<key>=<value>
php artisan feedback:bind-channel <channel_id> <destination> --tenant=<tenant>
```

`feedback-hub:install` publishes the config, the inspector script and the reports migration, then prints the Vite import and the component tag for your authenticated layout. The `mortelos-feedback-migrations` tag publishes the `central_channels` migration. The tenant `feedback_channel_bindings` migration ships in the package's `database/migrations/tenant` directory without a publish tag; copy it into your tenant migrations to use per-tenant bindings.

`feedback:central-channel` registers or updates the central default channel for a driver type and stores its credentials encrypted. `feedback:bind-channel` points one tenant at one of its own channels and replaces any earlier binding.

`feedback-hub:health`, `feedback-hub:test-delivery` (a dry run without `--send`), `feedback-hub:telegram-discover`, `feedback-hub:telegram-resolve` and `feedback-hub:telegram-test` set up and check the config-driven GitHub, Linear and Telegram handler.

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Blade component | `<x-feedback-hub::floating-button />` |
| Browser event that starts the flow | `open-feedback-hub` |
| Inspector script | `window.FeedbackHubInspector` |
| Submit route | `feedback-hub.store` |
| Admin routes | `feedback-hub.index`, `feedback-hub.show`, `feedback-hub.screenshot` |
| Delivery contract | `Mortel\Feedback\Contracts\FeedbackDeliveryHandler` |
| Default delivery handler | `Mortel\Feedback\Delivery\ChannelFeedbackDeliveryHandler` |
| Opt-in delivery handler | `Mortel\Feedback\Delivery\DefaultFeedbackDeliveryHandler` |

Pass `:floating="false"` to the component to hide the button and start the flow from your own control. Bind `DefaultFeedbackDeliveryHandler` to the contract to create GitHub and Linear issues and send a Telegram message from config instead of using channels.

## Configuration

`feedback-hub:install` publishes `config/feedback-hub.php`. The main switches are `enabled` (when off, the submit route returns 404), `project`, `route_prefix`, `route_middleware`, `admin_middleware`, `storage_disk`, `screenshot_path` and `queue`. The `github`, `linear` and `telegram` sections only feed the opt-in handler and the diagnostic commands; each has an `enabled` flag that is on by default, so turn off the ones you do not use.

## Boundaries

`mortelos/feedback` owns the widget, redaction, report storage, admin views, delivery job and channel resolution. `mortelos/framework` owns channels and the channel driver registry; the channel's driver does the sending. The host app owns access to the admin pages (the package adds nothing beyond `admin_middleware`), Alpine.js and a CSRF token (such as a `csrf-token` meta tag) in the layout, running each migration on the right connection, and channel credentials. The widget, admin pages and channel messages ship with Dutch copy. For annotating documents inside chat, use `mortelos/widget-document-feedback` instead.
