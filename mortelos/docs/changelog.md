---
title: "Changelog"
nav_title: "Changelog"
slug: "changelog"
version: "0"
description: "Release notes for MortelOS documentation version 0."
section: "reference"
order: 80
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/changelog"
last_verified: "2026-10-18"
public: true
---

# Changelog

## 0

2026-10-18:

1. Added [Mortel API v1](/docs/0/mortel-api) for framework 0.7.7 and 0.8.3, which are the same code. A host can switch on one versioned contract to read named queries, run public actions with a receipt and hear about changes live, over HTTP and MCP. The page covers switching it on, the routes, receipts, errors, live notifications, the `MortelServer` MCP server, adding your own queries and actions, and what the API refuses. The API is off until a host mounts it.
2. Added [Upgrading to 0.7.7](/docs/0/package-framework#upgrading-to-077): Laravel 13.25 or later, the `api_operations` migration only for an app that switches the API on, and webhook delivery that now refuses private, internal and link-local addresses and redirects to them.
3. Added the `MortelServer` mount to [MCP runtime](/docs/0/mcp-runtime#the-mortel-api-server) and pointed the [API](/docs/0/api) page to the new page.

2026-10-16:

1. Added [Coding guidelines](/docs/0/coding-guidelines): all code in packages and host apps is English, and Dutch stays in text a customer sees. The page covers the glossary rule, the exceptions, the `code-language: dutch-input` marker and the `CodeLanguageGuard` test. Added a Code language check to [Dev tools](/docs/0/package-dev-tools) and a Code Language section to [Agent standards](/docs/0/package-agent-standards).
2. Added an "Upgrading" section to the pages of the packages with breaking renames. These releases need a change in your app:

   - framework 0.7.0 (`php artisan migrate` after publishing the new migration), see [Upgrading to 0.7.0](/docs/0/package-framework#upgrading-to-070)
   - chat 0.6.0, overviews 0.4.0 and widget-compliance 0.4.0, which move together with framework 0.7
   - document-studio 0.4.0, channel-github 0.5.0 and sidecar 0.3.0, which require framework `^0.7`
   - triage 0.2.0 (0.2.1 allows framework 0.7)
   - channel-gmail 0.6.0 (0.6.1), channel-pocket 0.2.0 (0.2.1) and record 0.3.0 (0.3.1), which have no framework change
   - daily-planner 0.6.0 (0.6.1), which renames its events and DOM names

   Updated the names on the framework, overviews, widget-compliance, triage and sidecar pages. The record package has no page here yet; its upgrade note is in the package repository.

   Released in the same round, with no upgrade note: dev-tools 0.1.8, agent-standards 0.1.15, ui 0.5.2, entity-graph 0.5.1 and 0.5.2, policy-studio 0.3.3 and 0.3.4, widget-document-feedback 0.3.4 and 0.3.5, feedback 0.3.3 and 0.3.4, mail 0.1.4 and 0.1.5, issue-factory 0.1.4 and 0.1.5, social-login 0.2.2, app-standards 0.1.4, channel-fireflies 0.3.3 and 0.3.4, channel-google-calendar 0.4.2 and 0.4.3, channel-google-drive 0.5.1 and 0.5.2, channel-moneybird 0.4.7 and 0.4.8, channel-plaud 0.4.2 and 0.4.3, and channel-telegram 0.3.3 and 0.3.4.

3. Released starter 0.14.0 on framework 0.7. A new app now includes the `CodeLanguageGuard` test and the agent rules check in CI. Existing apps follow the steps in the starter's `UPGRADE.md`.

2026-10-14:

1. Added [Outlets: extend a package without changing it](/docs/0/outlets), with the five kinds of outlet, app examples, the seven-rung ladder, the checks, and guidance for package authors. Added an Outlets section to every package reference page, plus new pages for [Triage](/docs/0/package-triage) and the [GitHub channel](/docs/0/package-channel-github). Replaced the advice to publish package views with outlet or package pull request guidance.

   Verified against the outlets releases: dev-tools 0.1.7, ui 0.5.0, app-standards 0.1.3, agent-standards 0.1.14, framework 0.6.41, chat 0.5.0, overviews 0.3.1, entity-graph 0.5.0, document-studio 0.3.1, policy-studio 0.3.2, feedback 0.3.2, daily-planner 0.3.3, widget-compliance 0.3.2, widget-document-feedback 0.3.3, channel-gmail 0.5.0, channel-google-calendar 0.4.0, channel-google-drive 0.5.0, channel-github 0.4.0, channel-fireflies 0.3.2, channel-moneybird 0.4.6, channel-plaud 0.4.1, channel-pocket 0.1.4, channel-telegram 0.3.2, mail 0.1.3, issue-factory 0.1.3 and triage 0.1.2. Sidecar 0.2.5 ships its manifest too; its public page is outside this documentation round.

2026-09-26:

1. Added package pages for `mortelos/channel-google-calendar`, `mortelos/channel-pocket`, `mortelos/daily-planner`, `mortelos/document-studio`, `mortelos/feedback`, `mortelos/issue-factory` and `mortelos/policy-studio`, verified against their latest releases.
2. Listed the new pages and the existing app standards and agent standards pages in the navigation and the package overview.
3. Clarified that version 0 is written on `main`.

2026-06-07:

1. Updated installation docs for `mortelos v0.1.1` and the HTTPS starter clone default.
2. Added public build method, host app anatomy, TALL conventions, MCP runtime and troubleshooting pages.
3. Expanded starter, package governance, agentic development and first portal guidance.
4. Updated package reference pages with verified package details and current verification dates.

2026-05-31:

Initial public documentation structure:

1. Installation.
2. First portal.
3. Agentic development.
4. Package governance.
5. Package reference.
6. API reference shell.
