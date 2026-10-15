---
title: "Outlets: extend a package without changing it"
nav_title: "Outlets"
slug: "outlets"
version: "0"
description: "How your app plugs into a MortelOS package through its outlets, and what to do when no outlet fits."
section: "build-method"
order: 33
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/outlets"
last_verified: "2026-10-14"
public: true
---

# Outlets: extend a package without changing it

An outlet is a socket in a MortelOS package. The package decides where the sockets are and what fits into them; your app plugs into them. You use the declared extension points instead of patching or copying package code. After this page you know the five kinds of outlet, how to find the outlets of a package, how to plug into a screen, and what to do when no outlet fits.

## The idea in one minute

Say you want quick-reply buttons above the chat text field. You could copy the chat screen into your app and edit it. That works until the next `composer update`: then your copy is out of date and nobody notices. With an outlet you do something else. The chat package has marked a place above the text field, named `chat.composer.above-input`. Your app registers its own Blade view for that place, in one line in a service provider. The package stays untouched, updates keep working, and the check described below makes sure nobody takes the shortcut.

Every package lists its outlets in a file, `resources/mortelos/outlets.json`. If something is not in that list, it is not an outlet and you should not build on it.

## The five kinds of outlet

| Kind | The package offers | You do | With |
| --- | --- | --- | --- |
| contract | An interface or abstract class, usually with a default implementation | Bind your own class, or wrap the default | `$this->app->bind()` or `$this->app->extend()` |
| registry | A list you can add to: tools, widgets, menu items, step handlers | Register your item | `register()` on the registry, or a container tag |
| event | Something that happened | Listen to it | `Event::listen()` |
| config | A setting with a default value | Publish the config file and change it, or use `.env` | `php artisan vendor:publish --tag=<name>-config` |
| view | A place in a screen | Register your own Blade view, Livewire component or closure | `Outlet::view()`, `Outlet::livewire()`, `Outlet::render()` |

The first four are plain Laravel. Only the view outlet is new; it comes from `mortelos/ui` 0.5.0. The examples below use outlets that `mortelos/chat` 0.5.0 and `mortelos/framework` 0.6.41 really have. Put the code in the `boot()` method of one of your app's service providers, for example `app/Providers/AppServiceProvider.php`.

### Contract outlets

Chat decides which widget belongs to a message through the contract `WidgetMessageResolver`. To use your own rules, bind your own class:

```php
use Uteq\Chat\Contracts\WidgetMessageResolver;

$this->app->bind(WidgetMessageResolver::class, YourWidgetMessageResolver::class);
```

To keep the default and only add something before or after it, wrap it with `extend()`: `$this->app->extend(WidgetMessageResolver::class, fn ($default) => new YourWrapper($default));`. Your class implements the same interface and calls the default where it wants. Some contracts have no default: then the package says the host must bind it, or that the feature stays off until you do.

### Registry outlets

Chat keeps its widgets in `WidgetRegistry`. Add your own widget to it:

```php
use Uteq\Chat\Widgets\WidgetRegistry;

$this->app->extend(WidgetRegistry::class, fn (WidgetRegistry $registry) => $registry->register(new YourWidgetManifest));
```

Some registries are filled with a container tag instead of a method; the package page then says which tag.

### Event outlets

Chat declares no PHP event outlets in its current manifest. For an event example, the framework fires `AgentRunFinished` once when an agent run ends. React to it with a listener:

```php
use Illuminate\Support\Facades\Event;
use Mortel\Events\AgentRunFinished;

Event::listen(AgentRunFinished::class, YourListener::class);
```

### Config outlets

Chat keeps all its settings in `config/uteq-chat.php`. Publish that file into your app and change the copy:

```bash
php artisan vendor:publish --tag=uteq-chat-config
```

Never change package settings at runtime with `config()->set()` and never load the package's own config file with `require`: the check below fails on both.

### View outlets

A view outlet is a place in a package screen. Chat has four, among them `chat.composer.above-input` above the text field. See [Plug into a view outlet](#plug-into-a-view-outlet) below.

## Find the outlets of a package

`mortelos/dev-tools` 0.1.7 adds a command that lists the outlets of the packages installed in your app:

```bash
php artisan mortelos:outlets:list chat
php artisan mortelos:outlets:list chat --kind=view
php artisan mortelos:outlets:list chat --json
```

You can write the package name short (`chat`) or in full (`mortelos/chat`). Without a name you get every installed package. The table shows the package, the kind, the name, a description and, per kind, the default class, the registry method, the publish command or the data a view outlet passes. Each package reference page has an Outlets section with the same list as of the version it describes. Install `mortelos/dev-tools` with `composer require mortelos/dev-tools:^0.1.7 --dev` if the command is not available.

Run this command before you change anything in a package. That is also what the MortelOS agent rules ask an AI agent to do.

## Plug into a view outlet

A package marks a place in its screen. Whatever it shows there by default stays until someone plugs something in. Your app plugs in from a service provider. Choose one of the three registrations below; they are alternatives. Create your Blade views in `resources/views/chat/` (for example, `quick-replies.blade.php`), and register your Livewire component under the name used in the example.

```php
use Mortelos\Ui\Outlet;

public function boot(): void
{
    // A Blade view from your app: it receives $inputId
    Outlet::view('chat.composer.above-input', 'chat.quick-replies');

    // A Livewire component: it receives inputId as a mount parameter or property
    Outlet::livewire('chat.composer.above-input', 'chat-quick-replies');

    // A closure: it receives the data as an array and returns a view, an Htmlable or a string
    Outlet::render('chat.composer.above-input', fn (array $data) => view('chat.badge', $data));

    // Alternative: hide only the package's default
    Outlet::view('chat.composer.above-input', 'chat.quick-replies', replace: true);
}
```

- Order: first the package's default, then everything registered, in the order it was registered. Several packages and your app may use the same outlet.
- `replace: true` removes only the package's default, never what others registered.
- Data: the package passes data along, listed per outlet on its package page. The names arrive in camelCase, so `input-id` becomes `$inputId`. A Livewire component only receives the data it declares; extra objects are dropped and extra scalar values may become HTML attributes on the root element.
- A string returned from a closure is escaped. Return a view or an `Htmlable` for markup.
- `Outlet::has('chat.composer.above-input')` tells you whether anything is plugged in.

A typo in the name fails the check, with a suggestion:

```text
Outlet 'chat.composer.above-inputt' is not declared in any installed outlets manifest. Did you mean 'chat.composer.above-input' (mortelos/chat)? Run php artisan mortelos:outlets:list to see the outlets.
```

The full rules for the component are on the [UI package](/docs/0/package-ui) page.

## When no outlet fits

Walk down this ladder and stop at the first rung that fits.

| Rung | You want | Then |
| --- | --- | --- |
| 1 | A different value | Config outlet: publish the config file. |
| 2 | To react to something that happens | Event outlet: a listener in your app. |
| 3 | To add something to a list | Registry outlet: register it. |
| 4 | To replace or wrap a decision or behavior | Contract outlet: bind or wrap. |
| 5 | To add to a screen or replace a part of it | View outlet: `Outlet::view`, `Outlet::livewire` or `Outlet::render`. |
| 6 | Something no outlet covers | A pull request on the package that adds the missing outlet: small, additive, a patch release. Then use rungs 1 to 5 in your app. The behavior itself stays in your app. |
| 7 | Something that must hold for everyone | A pull request on the package with the behavior itself, and a package decision `build-in-mortelos` (see [Package governance](/docs/0/package-governance)). This is the right rung when two apps need the same thing, when it is a bug or a security fix, or when it is about the package's own default screens or default values. |

The difference between rung 6 and rung 7 in plain words: on rung 6 you add a socket to the wall and plug in at home; on rung 7 you change the appliance for everyone. Choose rung 6 when the behavior belongs only in your app; choose rung 7 when the change belongs in the package itself.

Never, at any rung: a patch on the package, a copy of a package view in `resources/views/vendor/`, a subclass of a concrete package class without a declared `swap` contract, `config()->set()` on a package setting, `require` of a package file in your config, a copy of a package screen in your app, or a fork of the package.

## How the check runs

`mortelos/dev-tools` 0.1.7 ships `OutletsGuard`, a test helper that fails your build when your app works around a package. Add one test next to `ArchGuardTest`, in a test folder your suite runs (in a starter app that is `tests/Feature/`):

```php
use Mortelos\DevTools\Testing\OutletsGuard;

it('extends mortelos packages only through their outlets', function () {
    OutletsGuard::assertClean(base_path());
});
```

It checks seven rules:

| Rule | Fails when | Do this instead |
| --- | --- | --- |
| `unknown-outlet` | `Outlet::view`, `Outlet::livewire` or `Outlet::render` uses a view name that neither an installed package nor your app declares in its manifest | Fix the name; the message suggests the closest one. |
| `concrete-subclass` | A class in your app extends a concrete (not abstract) class from a `mortelos/*` package without a declared `swap` contract | Bind or wrap the contract outlet, or open a package pull request (rung 6). |
| `config-set` | `config()->set()`, `Config::set()` or `config([...])` changes a package setting | Publish the config file and change it there, or use `.env`. |
| `vendor-require` | A scanned PHP file loads a file from `vendor/mortelos/` with `require` or `include` | Publish the package config and edit the copy. |
| `vendor-patch` | `.mortelos/package-patches.json`, a `patches/*.patch` file or `extra.patches` in `composer.json` changes a `mortelos/*` package | Use an outlet, or change the package itself. |
| `vendor-view-copy` | `resources/views/vendor/<name>` exists for a package's views, or for `mortelos-ui` | Register your own view in a view outlet. |
| `manifest` | An installed package ships an invalid `outlets.json` | Update the package; report it to the package if the latest release has the problem. |

The PHP rules scan `app/`, `bootstrap/` and `config/`, not `tests/` or `database/factories/`. A package without an `outlets.json` is a warning, not a failure, so an older dependency does not turn your build red.

An existing app may start with an allow list: per file the number of violations it has today, which may only go down, just like `ArchGuard`:

```php
OutletsGuard::assertClean(base_path(), allow: [
    'app/Providers/AppServiceProvider.php' => 2,
]);
```

To skip one PHP line on purpose, put `outlets-guard: ignore <reason>` in a comment on that line or on the line above it. Always write the reason.

For a quick look without a test, run:

```bash
php artisan mortelos:outlets:check
```

It runs the same scan without an allow list. The guard enforces the seven rules above; detecting a copied screen outside `resources/views/vendor/` or a fork still needs review. It exits with 1 when it finds a violation and with 0 when there are only warnings.

## For package authors

### Mark the spot

Mark each place in a package screen where a host may add something or replace a part with `<x-mortel::outlet>`. What stands between the tags is the default:

```blade
<x-mortel::outlet name="chat.composer.above-input" :input-id="$inputId">
    {{-- the default, often empty --}}
</x-mortel::outlet>
```

- Every attribute except `name` and `key` is data for the outlet.
- The tag adds no HTML element of its own; you choose the container.
- Inside a `@foreach`, pass `:key="$row->id"` so a Livewire component gets a stable key.
- Do not put an outlet inside `wire:ignore` or an `@island`.
- Require `mortelos/ui` `^0.5.0`.

### The manifest

Every package ships `resources/mortelos/outlets.json`. A shortened example:

```json
{
  "schema": "mortelos.outlets.v1",
  "package": "mortelos/chat",
  "views": ["uteq-chat"],
  "outlets": [
    {
      "kind": "view",
      "name": "chat.composer.above-input",
      "description": "Above the chat text field, for chips and quick replies.",
      "data": {"inputId": "string"},
      "default": "Empty"
    },
    {
      "kind": "contract",
      "name": "Uteq\\Chat\\Contracts\\WidgetMessageResolver",
      "description": "Decides which widget belongs to a chat message.",
      "default": "Uteq\\Chat\\Widgets\\DefaultWidgetMessageResolver"
    },
    {
      "kind": "registry",
      "name": "Uteq\\Chat\\Widgets\\WidgetRegistry",
      "method": "register",
      "description": "Add a WidgetDefinition or WidgetManifest."
    },
    {
      "kind": "config",
      "name": "uteq-chat",
      "tag": "uteq-chat-config",
      "description": "All keys in config/uteq-chat.php."
    }
  ]
}
```

| Field | Meaning |
| --- | --- |
| `schema` | Always `mortelos.outlets.v1`. |
| `package` | The same as `name` in the package's `composer.json`. |
| `views` | The Blade namespaces the package registers with `loadViewsFrom`; the check uses them to find copied views. |
| `outlets[].kind` | `view`, `contract`, `registry`, `event` or `config`. |
| `outlets[].name` | See naming rules below. |
| `outlets[].description` | One sentence for a reader who does not know the package. |
| `data`, `default` | For a view outlet: the data it passes (name and type) and what it shows without a registration. |
| `default` or `required` | For a contract: the default class; without one, `"required": true` (the host must bind it) or `"required": false` (unbound means the feature is off). |
| `swap` | For a concrete class that may be a contract (usually an Eloquent model): the config key where the host registers its subclass. |
| `method` or `tag` | For a registry: the method that registers, or the container tag. For a config outlet: the publish tag. |

A package without outlets ships `"outlets": []`: a promise that there is nothing to build on yet.

Browser events, Blade partials, gate abilities and JavaScript entry points do not fit schema v1. When a host may use them, the package page describes them in its Outlets section.

### The manifest test

One test in the package keeps the manifest and the code in line, in a test folder that `phpunit.xml` runs:

```php
use Mortelos\DevTools\Testing\OutletsManifest;

it('keeps outlets.json in line with the code', fn () => OutletsManifest::assertValid(dirname(__DIR__, 2)));
```

It fails when the file is missing or invalid, when a name appears twice, when a view outlet in the manifest has no `<x-mortel::outlet>` tag in `resources/views` or the other way round, when a contract is a concrete class without `swap` or its default does not implement it, when a registry class or method or an event class does not exist, and when a config outlet has no `config/<name>.php` or its tag does not appear in the package. Add the manifest entry in the same change as the outlet.

### Naming rules

- A view outlet is named `<package>.<screen>.<place>`: at least three parts, kebab-case, starting with the short package name (`chat.`, `entity-graph.`, `widget-compliance.`). Write it as a literal string, never built from variables.
- A contract, registry or event is named by its full class name, such as `Uteq\Chat\Contracts\WidgetMessageResolver`.
- A config outlet is named after the config file, such as `uteq-chat` for `config/uteq-chat.php`. New config files publish under the tag `<package>-config`.

## Versioning

| Change | Release |
| --- | --- |
| Add an outlet | Patch. Apps get it with their next `composer update` and nothing changes until they use it. |
| Raise the `mortelos/ui` floor to `^0.5` for the first view outlet | Minor. |
| Stop publishing a package's views | Minor. |
| Remove or rename an outlet | Breaking: a minor release while the package is below 1.0, with a line in the changelog. |

Package authors make concrete classes `final` when no host or test needs to extend or mock them. Hosts implement interfaces or extend abstract classes declared as contracts. A concrete model is an exception only when its contract entry has a `swap` key: register your subclass at that config key. A model without a declared `swap` is not an outlet.

## Words used on this page

| Word | Meaning |
| --- | --- |
| Package | A reusable piece of MortelOS code that you install with Composer, such as `mortelos/chat`. |
| Host app | Your Laravel app, which installs the packages. It starts from `mortelos/starter`. |
| Service provider | A class in `app/Providers/` whose `boot()` method runs when the app starts. That is where you plug into outlets. |
| Interface | A list of methods a class promises to have, without the code. |
| Contract | An interface or abstract class a package uses, so you can swap in your own class. |
| Bind | Tell Laravel which class to use when a package asks for a contract. |
| Registry | A list kept by a package that other code can add to. |
| Event | A message that something happened. A listener is code that runs when it arrives. |
| Config | A settings file in `config/`. |
| View | A Blade template that renders part of a screen. |
| Livewire component | A piece of screen with its own PHP state that updates without reloading the page. |
| Patch, minor, breaking | A patch is the last version number (0.5.1); a minor is the middle one (0.6.0). A breaking change requires existing apps to adapt. Below 1.0, MortelOS uses a minor release for breaking changes. |

## Related pages

- [Package governance](/docs/0/package-governance): package decisions, including `build-in-mortelos`.
- [Dev tools](/docs/0/package-dev-tools): `OutletsGuard`, `OutletsManifest` and the commands.
- [UI](/docs/0/package-ui): the `<x-mortel::outlet>` component and `Outlet`.
- [Host app anatomy](/docs/0/host-app-anatomy): where your code lives.
- [TALL conventions](/docs/0/tall-conventions): frontend and Laravel rules.
