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
last_verified: "2026-05-31"
public: true
---

# Installation

Create a new MortelOS Starter host app when you want a Laravel portal with the standard shell, authentication baseline, dashboard, inbox, governance, users and settings.

## Requirements

| Tool | Version or access |
| --- | --- |
| PHP | `^8.4` |
| Composer | `^2.7` |
| Node | `^20` |
| GitHub access | SSH or token access for private MortelOS packages when required |

## Create the host app

```bash
composer create-project mortelos/starter mijn-portal
cd mijn-portal
npm install --ignore-scripts
npm run build
php artisan starter:doctor
php artisan serve
```

Open `http://127.0.0.1:8000`.

The development seed account is:

| Email | Password |
| --- | --- |
| `admin@example.test` | `password` |

This account is only a local development baseline. Replace it before production use.

## Verify the baseline

Run the baseline checks before adding portal-specific behavior.

```bash
php artisan starter:doctor
vendor/bin/pest
```

Expected result:

1. Guests are redirected to `/login`.
2. Login works with the local seed account.
3. The default tenant is selected.
4. The dashboard loads.

