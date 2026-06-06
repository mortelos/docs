---
title: "Package Overview"
nav_title: "Overview"
slug: "packages"
version: "0"
description: "Reference index for public MortelOS packages."
section: "packages"
order: 50
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/packages"
last_verified: "2026-06-04"
public: true
---

# Package Overview

MortelOS follows the same documentation shape as Laravel: packages have their own navigation section, and every installable package gets a focused page. This page is the index.

Start with `mortelos/starter`, then add the package that matches the portal surface, integration channel or widget you need.

## Foundation

| Package | Type | Use when |
| --- | --- | --- |
| [`mortelos/starter`](/docs/0/starter-package) | Host app template | You need a new Laravel portal host with shell routes, auth baseline, dashboard, inbox, governance, users, settings and diagnostics. |
| [`mortelos/framework`](/docs/0/package-framework) | Core library | You need MortelOS primitives: entities, links, events, projections, tenant primitives, access resolution, MCP runtime and reusable application services. |
| [`mortelos/ui`](/docs/0/package-ui) | UI library | You need shared Flux-aligned UI primitives for a host app or reusable package surface. |

## Capability Surfaces

| Package | Type | Use when |
| --- | --- | --- |
| [`mortelos/chat`](/docs/0/package-chat) | Workspace package | You need the tenant-gated chat workspace, conversation panel and chat widget registry. |
| [`mortelos/overviews`](/docs/0/package-overviews) | Workspace package | You need reusable flexible overviews with datasource wiring, save-flow behavior, chat widgets and context integration. |
| [`mortelos/entity-graph`](/docs/0/package-entity-graph) | Workspace package | You need entity graph traversal, API routes, visualization, search, path-finding, chat widget support and agent tooling. |

## Channels

Channel packages connect MortelOS to external systems. Install them only when the portal needs that integration.

| Package | External system | Use when |
| --- | --- | --- |
| [`mortelos/channel-fireflies`](/docs/0/package-channel-fireflies) | Fireflies | You need inbound webhook handling and transcript ingest from Fireflies. |
| [`mortelos/channel-gmail`](/docs/0/package-channel-gmail) | Gmail | You need Gmail channel access for workspace communication or ingestion flows. |
| [`mortelos/channel-google-drive`](/docs/0/package-channel-google-drive) | Google Drive | You need Drive delivery for attachments, briefings or exported workspace material. |
| [`mortelos/channel-moneybird`](/docs/0/package-channel-moneybird) | Moneybird | You need scheduled invoice and contact sync from Moneybird. |
| [`mortelos/channel-plaud`](/docs/0/package-channel-plaud) | Plaud | You need Plaud channel ingestion for recorded or transcribed source material. |
| [`mortelos/channel-telegram`](/docs/0/package-channel-telegram) | Telegram | You need bidirectional Telegram Bot API communication. |

## Widgets

Widget packages add focused chat or workspace UI on top of the foundation packages.

| Package | Type | Use when |
| --- | --- | --- |
| [`mortelos/widget-compliance`](/docs/0/package-widget-compliance) | Chat widget package | You need compliance-focused chat widgets inside a MortelOS workspace. |
| [`mortelos/widget-document-feedback`](/docs/0/package-widget-document-feedback) | Chat widget package | You need document feedback flows inside the MortelOS chat surface. |

## Tooling

These packages support development and governance. They are not feature packages for every portal.

| Package | Type | Use when |
| --- | --- | --- |
| [`mortelos/dev-tools`](/docs/0/package-dev-tools) | Development tooling | You need Artisan commands for package decisions, governance checks and package-ready feature workflow support. |

## Package Boundaries

Use `mortelos/starter` for the host, `mortelos/framework` for core primitives, `mortelos/ui` for shared interface primitives and dedicated packages for reusable capabilities. Keep tenant-specific branding, policy defaults, local orchestration and one-off integrations in the host until they can serve more than one installation.

Package pages should describe stable contracts, not tenant-specific implementation details. Planned package names stay out of this index until source is available.
