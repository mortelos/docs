---
title: "MortelOS Daily Planner"
nav_title: "Daily planner"
slug: "package-daily-planner"
version: "0"
description: "Per-user daily planner with a backlog, time-boxed timeline, daily rituals and optional calendar push for MortelOS."
section: "packages"
order: 58
status: "mvp"
audience: "developers"
package: "mortelos/daily-planner"
canonical_path: "/docs/0/package-daily-planner"
last_verified: "2026-09-26"
public: true
---

# MortelOS Daily Planner

`mortelos/daily-planner` provides a per-user planner page with a backlog, a time-boxed day timeline and morning and evening rituals.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Backlog and today | Add tasks, move them between the backlog and today, mark them done and set minute estimates. Today shows estimated hours against tracked hours. |
| Timeline | A FullCalendar day view: drag tasks onto it to create time blocks, then move, resize or delete them. Quick-schedule books a task after today's last block, or at 09:00, for its estimate or 30 minutes. |
| Focus timer | Start and stop a timer on today's tasks; elapsed minutes are added to the task's tracked time. |
| Rituals | A morning dialog pulls selected backlog tasks into today with optional estimates. An evening shutdown records a mood and a reflection for the day. |
| Objectives and streams | Objectives for the current week with a done toggle, and colored streams that tag tasks. |
| Calendar | Show the user's calendar appointments read-only on the timeline, and push time blocks one-way to Google Calendar when the user opts in. Push is off by default. |
| Import | Import Gmail messages into the backlog as tasks, deduplicated per message. |
| Tenant storage | Tenant migrations for `planner_tasks`, `planner_time_blocks`, `planner_days`, `planner_objectives`, `planner_streams` and `planner_settings`. |

## Install

```bash
composer require mortelos/daily-planner
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The service provider is auto-discovered. It appends the package migrations to the host's `tenancy.migration_parameters` paths, so they run with the host's tenant migrations.

## Commands

```bash
php artisan planner:import gmail --user=<user-id>
php artisan vendor:publish --tag=daily-planner-views
php artisan vendor:publish --tag=daily-planner-migrations
```

`planner:import` accepts only the `gmail` source, which is the default, and requires `--user`. Views publish to `resources/views/vendor/daily-planner`; migrations publish to `database/migrations/tenant`.

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Route | `planner` (`GET /planner`, `web` and `auth` middleware) |
| Livewire page | `PlannerIndex` (view `daily-planner::livewire.pages.index`) |
| Sidebar item | `nav.sidebar.planner` |

The `Planner` sidebar item is registered only when the framework `NavigationItemRegistry` is bound. A tenant migration allows `nav.sidebar.planner` on the framework's default `Owner navigation access` and `Operator navigation access` policies.

## Boundaries

`mortelos/framework` owns policies and the sidebar registry. `mortelos/daily-planner` owns the planner data model, the `/planner` page and its task actions. All planner data is scoped to the signed-in user.

The host app owns tenancy initialization for `/planner`, because the route applies only `web` and `auth`. It also provides the `layouts::app` layout and the Flux components the page renders with, both of which ship with `mortelos/starter`. Bind channel-backed implementations of `GmailInboxReader`, `CalendarGateway` and `ChannelConnector` in the host. The package's null defaults import nothing, make no external calendar requests and list no connections.

In v0.1.1 the interface text is Dutch and cannot be translated; publish the views to change it. The timeline loads FullCalendar 6.1.15 from the jsDelivr CDN in the browser. Deleting a time block does not remove an event that was already pushed to the calendar.
