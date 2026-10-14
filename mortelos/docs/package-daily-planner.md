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
last_verified: "2026-10-14"
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
php artisan vendor:publish --tag=daily-planner-migrations
```

`planner:import` accepts only the `gmail` source, which is the default, and requires `--user`. Migrations publish to `database/migrations/tenant`.

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Route | `planner` (`GET /planner`, `web` and `auth` middleware) |
| Livewire page | `PlannerIndex` (view `daily-planner::components.pages.index`) |
| Sidebar item | `nav.sidebar.planner` |

The `Planner` sidebar item is registered only when the framework `NavigationItemRegistry` is bound. A tenant migration allows `nav.sidebar.planner` on the framework's default `Owner navigation access` and `Operator navigation access` policies.

## Outlets

An outlet is a place where you extend `mortelos/daily-planner` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.3.3; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list daily-planner
```

| Kind | Name | What it is for |
| --- | --- | --- |
| contract | `Uteq\DailyPlanner\Extras\DayExtras` | Adds your own rows to the Day board: at the top of the backlog, in a third column and as decoration on your own tasks. Default: `NullDayExtras`. |
| contract | `Uteq\DailyPlanner\Sync\CalendarGateway` | Reads and writes appointments in the user's external calendar. Default: `NullCalendarGateway`. |
| contract | `Uteq\DailyPlanner\Import\GmailInboxReader` | Reads incoming Gmail messages for the import into the backlog. Default: `NullGmailInboxReader`. |
| contract | `Uteq\DailyPlanner\Connect\ChannelConnector` | Connects the planner screen to the host's connect flow (Google Calendar, Gmail). Default: `NullChannelConnector`. |
| event | `Uteq\DailyPlanner\Events\TaskCompleted` | After a task is checked off. |
| event | `Uteq\DailyPlanner\Events\TaskReopened` | After a checked-off task is opened again. |
| config | `daily-planner` | The sidebar item: label and order (`navigation.label`, `navigation.sort`). Publish with `php artisan vendor:publish --tag=daily-planner-config`. |

## Boundaries

`mortelos/framework` owns policies and the sidebar registry. `mortelos/daily-planner` owns the planner data model, the `/planner` page and its task actions. All planner data is scoped to the signed-in user.

The host app owns tenancy initialization for `/planner`, because the route applies only `web` and `auth`. It also provides the `layouts::app` layout and the `mortelos/ui` components the page renders with, both of which ship with `mortelos/starter`. Bind channel-backed implementations of `GmailInboxReader`, `CalendarGateway` and `ChannelConnector` in the host. The package's null defaults import nothing, make no external calendar requests and list no connections.

The interface text is Dutch and cannot be translated yet. Do not copy the package views into your app to change it: `OutletsGuard` fails the build on a copied view (`vendor-view-copy`). The planner has no view outlets yet. To add your own rows to the Day board, bind the `DayExtras` contract; for anything else, open a pull request on the package that adds the outlet or the translation (see [Outlets](/docs/0/outlets#when-no-outlet-fits)). The timeline loads FullCalendar 6.1.15 from the jsDelivr CDN in the browser. Deleting a time block does not remove an event that was already pushed to the calendar.
