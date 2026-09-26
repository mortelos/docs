---
title: "MortelOS Issue Factory"
nav_title: "Issue factory"
slug: "package-issue-factory"
version: "0"
description: "Issue dossiers, issue storage and lifecycle commands for an autonomous TDD issue loop."
section: "tooling"
order: 67
status: "mvp"
audience: "developers"
package: "mortelos/issue-factory"
canonical_path: "/docs/0/package-issue-factory"
last_verified: "2026-09-26"
public: true
---

# MortelOS Issue Factory

`mortelos/issue-factory` provides the issue dossiers, issue storage and lifecycle commands that an autonomous TDD issue loop works from.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Issue dossiers | Describe one unit of work: title, problem, scope, acceptance criteria, a unit, feature, e2e and browser test plan, touched surfaces, package decision, priority, dependencies and risk. |
| Validation | Treat a dossier as incomplete without a title, problem, acceptance criteria, at least one test case or a package decision type. |
| Markdown feed | Write dossiers as markdown files with an embedded JSON block, by default in `.claude/issues`. |
| OS storage | Store dossiers as `issue` entities in a tenant. State changes, events, links and metrics are recorded as entity updates by the `issue-factory` actor. |
| Lifecycle | Track issues through `drafting`, `queued`, `in_progress`, `blocked`, `pr_ready`, `merged` and `cancelled`. |
| Ready queue | Pick the queued issue with the lowest `priority` value whose dependencies are all `merged`. |
| Agent skill | Ship a `create-issue` skill that turns a merged `mortelos/*` package change into dossiers through `issue-factory:create`. |

## Install

```bash
composer require mortelos/issue-factory
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

Register the `issue` entity type in each tenant that stores issues. The command does nothing when the type already exists:

```bash
php artisan issue-factory:install --tenant=<id|slug>
```

## Commands

```bash
php artisan issue-factory:create --input=issue.json --queue
php artisan issue-factory:sync --tenant=<id|slug> --dry-run
php artisan issue-factory:next --tenant=<id|slug> --peek --json
php artisan issue-factory:show <id> --tenant=<id|slug> --json
php artisan issue-factory:finish <id> --tenant=<id|slug> --pr=<pull-request-url>
php artisan issue-factory:finish <id> --tenant=<id|slug> --block --reason="Needs a host decision"
```

| Command | Behavior |
| --- | --- |
| `issue-factory:create` | Reads a JSON dossier from `--input` (a file, or `-` for stdin), stamps an id and writes it to the markdown feed as `drafting`, or `queued` with `--queue`. Refuses an incomplete dossier unless `--allow-incomplete` is passed. Needs no tenant. |
| `issue-factory:sync` | Ingests the markdown feed into the tenant. Adds only issues the tenant does not hold yet, promotes new `drafting` dossiers to `queued` and skips invalid ones. `--dry-run` reports without writing. |
| `issue-factory:next` | Shows the next ready issue and claims it as `in_progress`. `--peek` shows it without claiming. |
| `issue-factory:show` | Prints one issue. `--json` prints the full dossier. |
| `issue-factory:finish` | `--pr` moves an `in_progress` issue to `pr_ready` and links the pull request. `--block` with a required `--reason` moves it to `blocked`. Transitions the lifecycle does not allow are refused. |

`create` and `sync` accept `--path` to use another feed directory than the configured one.

## Configuration

Publish the config, or copy the `create-issue` skill to `.claude/skills/create-issue/SKILL.md` in the host:

```bash
php artisan vendor:publish --tag=issue-factory-config
php artisan vendor:publish --tag=issue-factory-skills
```

| Key | Default | Purpose |
| --- | --- | --- |
| `issue-factory.source` | `os` | `os` or `local`: which source the container binds for `IssueSource`. Env: `ISSUE_FACTORY_SOURCE`. |
| `issue-factory.local_path` | `.claude/issues` | Markdown feed directory, relative to the host base path. Env: `ISSUE_FACTORY_LOCAL_PATH`. |
| `issue-factory.entity_type` | `issue` | Entity type name used for issues in the tenant. |

Resolve `Mortelos\IssueFactory\Contracts\IssueSource` to read and write issues from host code. With `source` set to `os` it uses the tenant entity store when a tenant is initialised and falls back to the markdown feed otherwise. The artisan commands do not use this binding: `create` always writes the feed and the tenant commands always use the entity store.

## Boundaries

Use this package for the dossier format, issue storage, lifecycle states and the commands around them. It does not run the build loop, write code, or open or merge pull requests. The host owns the agent loop that claims issues with `issue-factory:next` and reports back with `issue-factory:finish`. No command sets `merged` or `cancelled`: the host records those, for example through `IssueSource::updateState()`, and an issue with dependencies only becomes ready once they are `merged`. Tenants and the entity store come from `mortelos/framework`.
