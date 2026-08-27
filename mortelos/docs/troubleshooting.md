---
title: "Troubleshooting"
nav_title: "Troubleshooting"
slug: "troubleshooting"
version: "0"
description: "Common MortelOS Starter installation and boot fixes."
section: "reference"
order: 75
status: "mvp"
audience: "developers"
package: "mortelos/starter"
canonical_path: "/docs/0/troubleshooting"
last_verified: "2026-08-27"
public: true
---

# Troubleshooting

Start every boot issue with:

```bash
php artisan starter:doctor
vendor/bin/pest
```

## `mortelos --version` is stale

Your shell is loading an older installed CLI before the current one.

```bash
type -a mortelos
mortelos --version
```

Install the current `bin/mortelos` into the first directory shown by `type -a`, or remove the stale copy. On macOS, `/opt/homebrew/bin` often appears before `/usr/local/bin`.

## GitHub SSH fails during `mortelos new`

`mortelos v0.1.1` clones `mortelos/starter` over HTTPS by default. If you still see:

```text
git@github.com: Permission denied (publickey).
```

you are running an older CLI. Reinstall the current script and run:

```bash
mortelos --version
```

It should return `mortelos v0.1.1` or newer.

## `mortelos/ui` cannot be installed

MortelOS packages come from the private registry at `https://packages.mortelos.com`, so a missing or wrong registry credential fails the install. A `401 Unauthorized` or a repeated credential prompt for `packages.mortelos.com` points at the credential; `Could not find a matching version` usually points at a missing repository entry.

Check both:

```bash
composer config --global --list | grep packages.mortelos.com
composer config repositories.mortelos
```

Set what is missing and install again:

```bash
composer config --global http-basic.packages.mortelos.com <customer> <token>
composer config repositories.mortelos composer https://packages.mortelos.com
composer install
```

GitHub SSH access is not involved; MortelOS packages are no longer installed from GitHub.

## Vite manifest not found

Frontend assets were not built.

```bash
npm install --ignore-scripts
npm run build
```

## `View [layouts.guest] not found`

The login page expects `resources/views/layouts/guest.blade.php`.

Restore it from Git:

```bash
git restore resources/views/layouts/guest.blade.php
```

## Missing starter route class config

If you see:

```text
LogicException: Missing starter route class config [...]
```

one of the route-backed auth config keys is `null` in `config/starter.php`. Fill the required class config keys or restore the starter defaults.

Required keys:

1. `auth.post_login_redirect_resolver`.
2. `auth.controllers.password_login`.
3. `auth.controllers.passkey_authenticated`.
4. `auth.controllers.accept_invitation`.
5. `users.resolver`.
6. `users.access_resolver`.

## Login loops back to login

Check:

1. After login, `Auth::check()` must be true.
2. `auth.post_login_redirect_resolver` must return a valid URL string.
3. The target route returned by the resolver exists.

## Sidebar, search or chat is empty

These are optional surfaces and degrade silently when their resolvers are `null`.

| Surface | Config to bind |
| --- | --- |
| Sidebar | `navigation.sidebar_resolver` |
| Universal search | `navigation.universal_search_resolver` |
| Governance | `governance.resolver` and `governance.access_resolver` |
| Users | `users.resolver` |
| Onboarding | `onboarding.resolver` |
| Inbox detail types | `inbox.item_type_resolver` |
| Chat panel | `chat.settings_service` and `chat.conversation_panel_component` |

Bind these only when the capability map calls for them.
