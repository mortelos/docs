---
title: "Installation"
nav_title: "Installation"
slug: "installation"
version: "0"
description: "Install a MortelOS Starter host app and verify the boot baseline."
section: "getting-started"
order: 10
status: "mvp"
audience: "developers"
package: "mortelos/starter"
canonical_path: "/docs/0/installation"
last_verified: "2026-08-27"
public: true
---

# Installation

Create a new MortelOS Starter host app when you want a Laravel portal with the standard shell, authentication baseline, dashboard, inbox, governance, users and settings.

## Meet MortelOS

MortelOS is a Laravel portal system for building governed customer portals from a stable host application and reusable capability packages.

The starter app gives every installation the same baseline: authentication, dashboard shell, inbox, governance surfaces, users and settings. Project-specific behavior starts in the host and moves into packages when it can serve more than one installation.

### Why MortelOS?

MortelOS keeps the first portal fast without making the second portal expensive. The starter app handles the recurring operational surface, while packages keep reusable domain behavior portable across installations.

### An Agent Ready Portal System

MortelOS is built for human-directed, agent-assisted development. File locations, package boundaries, policy rules and review checkpoints are documented so an AI coding agent can work inside predictable constraints.

Read [Agentic Development](/docs/0/agentic-development) before asking an agent to build the first customer-specific capability.

## Creating a MortelOS Host Application

### Getting Started Using AI

If you use an AI coding agent, start with a prompt that gives it the MortelOS playbook before it touches the project:

If your machine cannot yet create, boot or verify a MortelOS host app, use the `mortelos-tooling-setup` skill before `setup-portal`. Trigger it for missing Herd, PHP, Composer, Node, GitHub access, MortelOS CLI, DBngin, TablePlus or `mortelos new` setup.

```text
I'm building a new MortelOS portal host application.

Read and follow the MortelOS installation guide:
https://mortelos.nl/docs/0/installation

Then read the agentic development guide:
https://mortelos.nl/docs/0/agentic-development

If the machine is not ready yet, use the mortelos-tooling-setup skill first.
Create the host with mortelos new, keep Laravel defaults where MortelOS does not specify otherwise, and stop before adding customer-specific behavior.
```

After the agent creates the host app, continue with a capability-first interview before implementation work starts.

### Installing PHP and Composer

| Tool | Version or access |
| --- | --- |
| PHP | `^8.4` |
| Composer | `^2.7` |
| Node | `^20` |
| MortelOS package access | A customer name and token for `https://packages.mortelos.com` |

If you do not have a local PHP stack yet, install PHP and Composer first. Laravel Herd is the fastest path on macOS and Windows.

### Installing the MortelOS CLI

The starter ships a small CLI script at `bin/mortelos`. Install it once from a trusted starter checkout. The current CLI version is `v0.1.1`.

```bash
git clone https://github.com/mortelos/starter.git mortelos-starter
cd mortelos-starter
mkdir -p ~/.local/bin
install -m 0755 bin/mortelos ~/.local/bin/mortelos
mortelos --version
```

`~/.local/bin` is the preferred local install target because it does not require administrator permissions. Make sure that directory is in your `PATH` before older system-wide install paths.

For a system-wide install, use `/usr/local/bin` only when you can write to it. On macOS with Homebrew, `/opt/homebrew/bin` usually appears before `/usr/local/bin` and is often the better user-writable target:

```bash
install -m 0755 bin/mortelos /usr/local/bin/mortelos
# or
install -m 0755 bin/mortelos /opt/homebrew/bin/mortelos
```

If that command returns `Permission denied`, install into `~/.local/bin` instead or rerun the system-wide install with administrator permissions.

If `mortelos --version` still shows an older version, inspect every matching binary:

```bash
type -a mortelos
```

Install the new script into the path that appears first, or remove the stale copy.

The CLI uses `https://github.com/mortelos/starter.git` by default. Override it with `MORTELOS_STARTER_REPO` or `MORTELOS_STARTER_BRANCH` when you need another source or branch.

### Configuring Package Access

MortelOS packages are distributed from the private Composer registry at `https://packages.mortelos.com`, not from GitHub. Composer needs one repository entry and one credential before it can install `mortelos/*` packages. Set the credential on your machine; check the repository entry in the host app and add it when it is missing.

Configure the credential once per machine with the customer name and token you received:

```bash
composer config --global http-basic.packages.mortelos.com <customer> <token>
```

That writes to your global `auth.json` and applies to every host app on the machine. In CI, pass the same credential through an environment variable instead of committing it:

```bash
composer config --global http-basic.packages.mortelos.com <customer> "$MORTELOS_REGISTRY_TOKEN"
```

If a host app does not have the repository entry yet, add it:

```bash
composer config repositories.mortelos composer https://packages.mortelos.com
```

Host apps require MortelOS packages by version range (`"mortelos/framework": "^0.6"`), never as `@dev` and never through a `path` repository. See [Packages](/docs/0/packages) for how to work on a package locally.

### Creating the Host Application

Use the CLI when it is available:

```bash
mortelos new mijn-portal
cd mijn-portal
composer dev
```

`mortelos new` shallow-clones the starter, removes the starter Git history, initializes a fresh repository and runs `composer setup`.

During `composer setup`, Composer downloads `mortelos/ui`, `mortelos/framework` and the other MortelOS packages from `https://packages.mortelos.com`. Configure the registry credential first, otherwise this step fails with an authentication error. Cloning the starter itself needs no GitHub access.

If the CLI is not installed yet, use Composer directly:

```bash
composer create-project mortelos/starter mijn-portal
cd mijn-portal
npm install --ignore-scripts
npm run build
php artisan starter:doctor
php artisan serve
```

Open `http://127.0.0.1:8000`.

`composer setup` and `composer create-project` both install the PHP dependencies and run the starter bootstrap hooks. That means `vendor/`, `.env`, the SQLite database file, migrations and the development seed account are already in place before the Vite build runs.

The frontend build imports Livewire and Flux assets from Composer packages under `vendor/`. If you created the app from an existing checkout or copied the files manually, run the Composer bootstrap first:

```bash
composer install
php -r "file_exists('.env') || copy('.env.example', '.env');"
php artisan key:generate
php -r "file_exists('database/database.sqlite') || touch('database/database.sqlite');"
php artisan migrate --force
php artisan db:seed --force
npm install --ignore-scripts
npm run build
```

## Initial Configuration

### Environment Based Configuration

Review `.env` before adding customer data or external integrations. Set the application URL, mail transport, queue connection and database connection for the target environment.

Local development can use the starter defaults. Production and staging should use explicit environment values managed outside Git.

### Databases and Migrations

Run the starter migrations before the first browser check:

```bash
php artisan migrate
```

Use SQLite for quick local verification when the project has no database choice yet. Switch to MySQL or PostgreSQL when the customer environment requires it.

### Tenant and User Baseline

The starter creates a default tenant and a local development seed account:

| Email | Password |
| --- | --- |
| `admin@example.test` | `password` |

This account is only a local development baseline. Replace it before production use.

A row in `users` is an identity, not a tenant membership. The `tenant_user` row
links that identity to the active tenant and carries its tenant role. The seed
creates both records for the admin account.

When `tenant_user` exists, MortelOS requires an explicit row for the active
tenant. Creating or importing a user without that row does not grant access.
The framework reads `role_user` only as a compatibility fallback for older
installations where the `tenant_user` table itself does not exist.

## Installation Using Herd

Laravel Herd provides PHP, Nginx, Composer and Node tooling for local Laravel development. MortelOS host apps work well in a Herd parked directory.

### Herd on macOS

Create the host app inside your parked directory:

```bash
cd ~/Herd
mortelos new mijn-portal
cd mijn-portal
herd open
```

Run the baseline checks after the app opens.

### Herd on Windows

Create the host app from PowerShell inside the Herd parked directory:

```powershell
cd ~\Herd
mortelos new mijn-portal
cd mijn-portal
herd open
```

Use the Herd UI to confirm the PHP version and site domain.

## IDE Support

Use an editor that can follow Laravel conventions, Blade views, Livewire components, package source paths and tests. VS Code, Cursor and PhpStorm all work well for MortelOS projects.

For agent-assisted work, keep `AGENTS.md`, the project README and the MortelOS docs open as source material.

## MortelOS and AI

MortelOS agents should work from accepted scope, not from guesses. Start by mapping roles, data, actions, approvals and package boundaries.

### Installing Agent Guidance

Keep the project guidance files in the host app and update them when the portal introduces new package rules, naming conventions or customer-specific constraints.

Use the [Agentic Development](/docs/0/agentic-development) guide as the baseline for prompts and review checkpoints.

## Verify the Baseline

Run the baseline checks before adding portal-specific behavior.

```bash
php artisan starter:doctor
vendor/bin/pest
```

Expected result:

1. Guests are redirected to `/login`.
2. Login works with the local seed account.
3. The user is redirected to `/dashboard`.
4. Users, event store and starter config checks pass.

## Next Steps

Now that the host app is running, continue with the first customer-facing vertical slice:

1. Build the first portal flow with [First Portal](/docs/0/first-portal).
2. Review package ownership with [Package Governance](/docs/0/package-governance).
3. Check reusable starter capabilities in [Starter Package](/docs/0/starter-package).
