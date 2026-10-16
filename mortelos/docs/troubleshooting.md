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
last_verified: "2026-10-16"
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

No customer name and token yet? See [Requesting Package Access](/docs/0/installation#requesting-package-access).

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

These are optional surfaces. Their resolvers are `null` in a fresh starter, and the surface stays off until you bind one.

| Surface | Config to bind |
| --- | --- |
| Sidebar | `navigation.sidebar_resolver` |
| Universal search | `layout.universal_search_component` and `navigation.universal_search_resolver` |
| Governance | `governance.resolver` and `governance.access_resolver` |
| Users | `users.resolver` |
| Onboarding | `onboarding.resolver` |
| Inbox detail types | `inbox.item_type_resolver` |
| Chat panel | `chat.settings_service` and `chat.conversation_panel_component` |

Bind these only when the capability map calls for them.

`php artisan starter:doctor` warns about the three that a fresh starter runs into first. A warning does not fail the doctor:

| Config | While it is `null` |
| --- | --- |
| `navigation.sidebar_resolver` | The sidebar stays empty. |
| `onboarding.resolver` | `/onboarding` shows an empty state that names this key instead of the welcome steps. |
| `inbox.item_type_resolver` | The inbox opens every item in the approval flow, never in the intake pane. |

### Fill the sidebar

The sidebar needs two keys: `layout.sidebar_nav_component` set to `shared.sidebar-nav`, and `navigation.sidebar_resolver` set to a host class with three methods:

1. `sections($user)` returns the menu sections.
2. `inboxCount($user)` returns the badge count on the inbox item.
3. `overviews($user)` returns the saved overviews, or `[]`.

For `sections()`, return `app(Mortel\Support\NavigationRegistry::class)->forUser($user)` from `mortelos/framework`. The registry is named `forUser()`, not `sections()`, so point the key at your own class and call the registry from there.

`forUser()` filters every item through a policy with scope `navigation` and denies by default, so it returns an empty list until the user's role has such a policy allowing the built-in item permissions: `nav.sidebar.inbox`, `nav.sidebar.search`, `nav.sidebar.dashboard`, `nav.sidebar.governance`, `nav.sidebar.users` and `nav.sidebar.apps`. From `mortelos/framework` 0.6.39 the registry leaves out any item whose route the host does not define. On older versions, leave `nav.sidebar.apps` out unless the host defines an `apps.marketplace` route, because the sidebar cannot link to a route that does not exist.

### Fill universal search

Universal search is the search window that opens with ⌘K, or Ctrl+K on Windows and Linux. It needs two keys in `config/starter.php`:

```php
'layout' => [
    // ...
    'universal_search_component' => 'shared.universal-search',
],

'navigation' => [
    // ...
    'universal_search_resolver' => App\Support\UniversalSearchResolver::class,
],
```

While `layout.universal_search_component` is `null`, the layout does not render the component and ⌘K does nothing. While `navigation.universal_search_resolver` is `null`, the window opens but shows no results. The layout hides the component's own search button, so the window opens only with ⌘K or the browser event `open-universal-search`, for example from a sidebar item with `'type' => 'action'` and `'action' => 'open-universal-search'`.

The resolver needs no interface. The component resolves the class from the container and calls three methods on it, with `$user` as `auth()->user()`:

1. `results(string $query, $user)` returns an array with the keys `entities`, `inboxItems` and `chatItems`, each a list of items. The component calls it with `''` when the page loads and again while the user types, with the query trimmed and cut to 120 characters. It does not match the lists to the query itself, so that is the resolver's job. A missing key counts as an empty list.
2. `navigation($user)` returns the pages to jump to. The component calls it once when the page loads and filters it on `label` in the browser.
3. `saveOverview(string $query, string $name, int|string $userId)` runs when the user saves the query with "Bewaar als overzicht", with `auth()->id()` as `$userId`. The component ignores the return value and then dispatches `overzicht-saved`, on which `shared.sidebar-nav` reloads `overviews($user)` from the sidebar resolver. The button shows whenever there is a query and at least one entity result, so keep the method even when it does nothing.

Every item is an array of strings:

| List | Keys | The window shows |
| --- | --- | --- |
| `entities` | `id`, `name`, `type`, `url`, `icon` | `name` and `type`; the first five on an empty query. |
| `inboxItems` | `id`, `title`, `summary`, `type`, `status`, `url`, `icon` | `title`, `summary` and `status`. |
| `chatItems` | `id`, `title`, `summary`, `url`, `icon` | `title` and `summary`. |
| `navigation()` | `label`, `icon`, `route` | `label`. |

Choosing an item opens its `url` with `Livewire.navigate()`; a navigation item opens its `route`. So `route` holds a URL such as `route('settings')`, not a route name. `icon` is a name for `<x-mortel::icon>`, such as `magnifying-glass`, and `id` is unique within its list. The component sends the lists to the browser as they are, so scope them to the user and tenant in the resolver.

A minimal resolver:

```php
<?php

declare(strict_types=1);

namespace App\Support;

use Illuminate\Contracts\Auth\Authenticatable;

final class UniversalSearchResolver
{
    /**
     * @return array{entities: list<array<string, string>>, inboxItems: list<array<string, string>>, chatItems: list<array<string, string>>}
     */
    public function results(string $query, ?Authenticatable $user): array
    {
        // Replace with your own query, scoped to the user and tenant.
        $entities = [
            ['id' => '1', 'name' => 'Acme', 'type' => 'Customer', 'url' => '/customers/1', 'icon' => 'building-office'],
        ];

        return [
            'entities' => array_values(array_filter(
                $entities,
                fn (array $entity): bool => $query === '' || str_contains(mb_strtolower($entity['name']), mb_strtolower($query)),
            )),
            'inboxItems' => [],
            'chatItems' => [],
        ];
    }

    /**
     * @return list<array{label: string, icon: string, route: string}>
     */
    public function navigation(?Authenticatable $user): array
    {
        return [
            ['label' => 'Dashboard', 'icon' => 'home', 'route' => route('dashboard')],
            ['label' => 'Instellingen', 'icon' => 'cog-6-tooth', 'route' => route('settings')],
        ];
    }

    public function saveOverview(string $query, string $name, int|string $userId): void
    {
        // Store the overview through a host Action, or leave this empty.
    }
}
```
