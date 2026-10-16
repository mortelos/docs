---
title: "GitHub Channel"
nav_title: "GitHub channel"
slug: "package-channel-github"
version: "0"
description: "GitHub App channel that brings pull requests into the MortelOS inbox."
section: "channels"
order: 65
status: "mvp"
audience: "developers"
package: "mortelos/channel-github"
canonical_path: "/docs/0/package-channel-github"
last_verified: "2026-10-16"
public: true
---

# GitHub Channel

`mortelos/channel-github` brings GitHub pull requests into the MortelOS inbox through a GitHub App, with approval-bound documentation publication and explicit merge or close decisions.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Pull requests in the inbox | Poll the repositories of a connection every five minutes and keep one inbox card per pull request up to date, including merges and closes done on GitHub. |
| Pull request details | Show state, head and base, checks, statuses and the escaped diff inside an inbox detail the host has already authorized. |
| Merge and close | Let a user merge or close a pull request from the inbox, only with a fresh, complete snapshot and the host's permission. |
| Publication | Turn an approved publication proposal into a pull request, through a workflow step. |
| Connector setup | Register the GitHub connector so a user can set up a connection. |
| Tenant storage | Migrations for the pull request projection, polling cursors, publication proposals and pull request decisions. |

## Install

```bash
composer require mortelos/channel-github
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The service provider is auto-discovered. Add the package migration folder to the host's tenant migration paths, next to the framework migrations. The host must also bind the framework `TenantResolver` to its own tenancy and bind `GitHubCredentialResolver` (see Outlets below), and run its scheduler and queue workers.

## Commands

```bash
php artisan channel:poll-github --tenant=<id-or-slug>
php artisan channel:recover-github-publications
php artisan channel:recover-github-decisions
```

All three run every five minutes through the Laravel scheduler. Without `--tenant`, polling covers every tenant.

## Upgrading to 0.5.0

Require `mortelos/framework:^0.7` alongside this package and follow the framework 0.7 migration steps; see [Upgrading to 0.7.0](/docs/0/package-framework#upgrading-to-070). If the host publishes `config/mortel.php`, rename `inbox.max_per_bron_per_dag` to `inbox.max_per_source_per_day`.

## Outlets

An outlet is a place where you extend `mortelos/channel-github` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.4.0; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list channel-github
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| config | `channel-github` | The channel used to create publication proposals (`publication_channel_id`). Publish with `php artisan vendor:publish --tag=channel-github-config`. |  |
| contract | `Uteq\GitHubChannel\Contracts\GitHubCredentialResolver` | Supplies the GitHub App details of the chosen connection from the host's encrypted storage. No default: the host must bind it. |  |
| contract | `Uteq\GitHubChannel\Contracts\PublicationChannelResolver` | Decides which channel a publication proposal uses; by default the channel from `channel-github.publication_channel_id`. Default: `ConfiguredPublicationChannel`. |  |
| view | `channel-github.pull-request.summary` | At the top of a pull request in the inbox, below the title. Default: nothing. | `pullRequest`, `title` |
| view | `channel-github.publication.summary` | At the top of a publication proposal, above the warning that it becomes public. Default: nothing. | `proposal`, `execution` |

Merging and closing a pull request from the inbox check the gate abilities `github_pr.merge` and `github_pr.close`; define them in the host. Render the pull request and publication details inside your own authorized inbox detail with `PullRequestDetails::render()` and `PublicationDetails::render()`. Gates and these render helpers are not outlets in the manifest.

## Boundaries

The package owns the GitHub App sign-in, the read-only polling, the inbox cards, the publication flow and the merge and close decisions. The host app owns the GitHub App credentials and where they are stored, which repositories a connection may read, tenancy, and who may merge or close. The package stores no secrets and has no environment variables for credentials.
