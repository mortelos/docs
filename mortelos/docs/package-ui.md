---
title: "MortelOS UI"
nav_title: "UI"
slug: "package-ui"
version: "0"
description: "The MortelOS component kit: x-mortel:: Blade components for host apps and packages, with one shadcn-style implementation and no Flux."
section: "packages"
order: 53
status: "mvp"
audience: "developers"
package: "mortelos/ui"
canonical_path: "/docs/0/package-ui"
last_verified: "2026-10-28"
public: true
---

# MortelOS UI

`mortelos/ui` provides shared interface primitives for MortelOS host apps and reusable packages.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Components | Every screen of a host app or package is built from `<x-mortel::…>` Blade components: buttons, fields, selects, tables, menus, dialogs, toasts, the app frame, cards, tabs, key figures, page headers and action bars, lists, a command palette, steps and a status board. |
| Own kit | The one implementation of those components, with the look of shadcn/ui. Since 0.6.0 there is no Flux and no switch. |
| Theme | One set of semantic tokens (`--primary`, `--muted`, `--border`, `--radius`, …) that apps set to taste. |
| Guards | `UiGuard` keeps screens on the kit and Flux-free; `LivewireMonitor` budgets each interaction in browser tests. |
| View outlets | `<x-mortel::outlet>` marks a place in a package screen; `Mortelos\Ui\Outlet` lets an app plug its own view, Livewire component or closure into it. |

## Install

```bash
composer require mortelos/ui
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## The own kit

`mortelos/ui` has one implementation of every component that host apps and packages use: Blade components with the look of shadcn/ui (through Basecoat, MIT), on Alpine and the browser's own `<dialog>` and popover. Apps write `<x-mortel::…>` tags, as they always did. Since 0.6.0 Flux is gone: `livewire/flux` is no longer a dependency, and `MORTEL_UI_KIT` and `config('mortel-ui.kit')` do nothing.

- Own components are at least as fast as the Flux wrappers they replaced, in CPU time and in HTML bytes; the package's CI measures it for every component.
- Tags that only existed for Flux are removed in 0.6.0 (navbar, navlist, otp, toggle and others). The migration guide lists them with the component to use instead.
- The kitchen sink at `/_mortel` (environment `local`) shows every component.
- `main` is the 0.6.x release line. `release/0.5` exists only when an app still on 0.5 needs a fix; it is cut from the last 0.5 tag and tagged `v0.5.N`.

## Documentation for AI agents

The package documents itself for an agent that builds a screen and must not invent anything. In an installed app it lives under `vendor/mortelos/ui/`. This set of files, `UiGuard::assertFluxFree()` and `UiGuard::assertAppComponents()` ship from 0.5.14 and stay in 0.6.

| File | What it holds |
| --- | --- |
| `COMPONENTS.md` | The one entry point: the rules (kit first, no UI components of an app's own, ask the kit for a missing piece), a decision table from need to component, theming, the speed rules, the guards and every tag with its implementation. |
| `docs/components/<family>.md` | One page per family: the exact props and values, slots, events and hooks of each component, a minimal and a full example, and the mistakes to avoid. |
| `docs/recipes.md` | Copy-ready screens: a list page with filters, table, empty state and paging; a detail page with an inspector; a form with page actions; a settings page; a dashboard; a confirm dialog; a toast with undo; loading placeholders. |
| `docs/theme.md` | The theme tokens and how to set the brand. |
| `docs/migration.md` | How an app moves from 0.5 to 0.6, and from Flux to the own kit. |

A doc-drift test in the package fails when a component or prop has no documentation, when an example or recipe does not render, when a recipe does not pass `UiGuard`, or when the decision table names a component that does not exist. The agent rules of [`mortelos/agent-standards`](/docs/0/package-agent-standards) ("TALL And UI") point agents to `COMPONENTS.md` first.

An app may have Blade components of its own for what is specific to it, on the kit and under the kit's rules: only `<x-mortel::…>` tags and HTML, no network calls, no server calls of their own, no scripts, no Flux, and field roles as props rather than business names. `UiGuard::assertAppComponents()` checks that.

## Moving an app to 0.6

`vendor/mortelos/ui/docs/migration.md` is the guide. Which steps you need depends on where the app stands.

**An app on 0.5 in own mode** ("From 0.5 to 0.6" in the guide):

1. Widen `mortelos/ui` to `^0.6` in the app and in its packages, and update together. A package that still requires `^0.5` blocks the upgrade.
2. Remove `MORTEL_UI_KIT` from environment examples, deploy configuration, test slots and CI, and the `kit` key from a published `config/mortel-ui.php`.
3. Run `UiGuard::assertFluxFree()`. When it is green, drop `livewire/flux` from the app's `composer.json` and run `composer update`. Remove `Flux\FluxServiceProvider` and every `use Flux\…` import.
4. Run `php artisan view:clear` once on the deploy. Compiled views move back to the configured `view.compiled` folder.
5. The Livewire component macro `$this->modal('x')->show()` and `->close()` stops working without Flux. Use `$this->dispatch('modal-show', name: 'x')` and `'modal-close'`.
6. Drop the `flux` CI leg. Run the suites once.

**An app still on Flux** follows the Flux to kit steps of the guide and lands directly on 0.6: `@mortelAppearance` and `@mortelScripts` in the layouts instead of `@fluxAppearance` and `@fluxScripts`; `flux.css` out and `kit.css` in; `Flux::modal(…)` becomes `$this->dispatch('modal-show', name: …)` and `'modal-close'`; toasts use the `mortel-toast` event; `$flux.appearance` becomes `$store.mortel.appearance`; app CSS, JavaScript and tests that select `data-flux-*` hooks select the kit hooks (`data-mortel-*`); app icons move to `resources/views/mortel/icon/`. `bin/flux-to-mortel.php` converts the Blade tags it can.

Both end at the same check: `UiGuard::assertFluxFree()` fails on any Flux left in Blade, PHP, CSS and JavaScript.

```php
use Mortelos\Ui\Testing\UiGuard;

it('keeps the UI on the kit', function () {
    UiGuard::assertClean(resource_path('views'), app_path('Providers'));
    UiGuard::assertAppComponents(resource_path('views/components'));
    UiGuard::assertFluxFree(resource_path('views'), resource_path('css'), resource_path('js'), app_path());
});
```

The guide holds the table from every `data-flux-*` hook and Flux variant to its kit hook.

## Theme

Import `kit.css` after Tailwind and define the app's dark variant. Its tokens follow shadcn/ui's names (`--background`, `--primary`, `--muted`, `--border`, `--radius`, …); set them on `:root` and `.dark` after the import to change the look. An app that sets `--color-accent`, `--color-accent-content` and `--color-accent-foreground` keeps its brand: the kit reads them. The full list is in `vendor/mortelos/ui/docs/theme.md`.

```css
@import 'tailwindcss';
@import '../../vendor/mortelos/ui/resources/css/kit.css';
@custom-variant dark (&:where(.dark, .dark *));
```

The starter derives its brand from one token through the optional MortelOS brand layer `mortel.css`, imported before the kit:

```css
@import 'tailwindcss';
@import '../../vendor/mortelos/ui/resources/css/mortel.css';
@import '../../vendor/mortelos/ui/resources/css/kit.css';

@theme {
    --mortel-accent: #1466ff;
}
```

One token is enough. The layer derives readable button text and link tints for light and dark mode. Without any `--mortel-*` token the layer sets zinc-800 in light mode and white in dark mode. Without `mortel.css` the kit's own defaults (zinc-900 in light, zinc-200 in dark) apply. In 0.6 `mortel.css` holds only this derivation; its old rules for Flux components are gone.

| Token | Used for | Default |
| --- | --- | --- |
| `--mortel-accent` | Primary buttons, active navigation, accent surfaces | zinc-800 in light, white in dark |
| `--mortel-accent-content` | Accent-coloured text in light mode | Derived: brand colour, darkened when needed to stay readable on white |
| `--mortel-accent-foreground` | Text on accent surfaces in light mode | Derived: white or black, whichever contrasts more |
| `--mortel-accent-dark` | Accent in dark mode | `--mortel-accent` |
| `--mortel-accent-content-dark` | Accent-coloured text in dark mode | Derived: brand colour, lightened when needed to stay readable on dark |
| `--mortel-accent-foreground-dark` | Text on accent surfaces in dark mode | Derived: white or black, whichever contrasts more |

- Set the tokens in `@theme` or on `:root`. On a deeper element they have no effect, because the derivation happens on `:root`.
- Import the brand layer before `kit.css`, so the kit reads the derived values.
- Do not set `--color-accent` yourself or add your own `.dark` rule for it; the layer handles both modes.
- Neutral greys follow Tailwind's zinc: keep the app's `--color-zinc-50` to `--color-zinc-950` in the same `@theme` block. The kit resets no zinc shades. For example, `--color-zinc-100` sets light muted, secondary and hover surfaces, and `--color-zinc-800` sets them in dark mode.
- A typo in a colour value makes the accent invalid (transparent buttons). Check a primary button in light and dark after changing it.
- The derivation uses CSS relative colour syntax and `round()`. It is tested in Chromium, WebKit and Firefox.

## View outlets

A view outlet is a place in a package screen where your app can add something, or replace the part the package shows there, without copying the package's views. It ships with `mortelos/ui` 0.5.0. [Outlets](/docs/0/outlets) explains the idea and the other four kinds of outlet.

A package marks the place in its Blade view. What stands between the tags is the default; it stays until someone plugs something in:

```blade
<x-mortel::outlet name="chat.composer.above-input" :input-id="$inputId">
    {{-- the package's default, often empty --}}
</x-mortel::outlet>
```

Your app plugs in from the `boot()` method of a service provider. Choose one registration below; they are alternatives. The Blade view `chat.quick-replies` lives in your app's `resources/views/chat/quick-replies.blade.php`. Use the name you registered for your Livewire component:

```php
use Mortelos\Ui\Outlet;

public function boot(): void
{
    // A Blade view
    Outlet::view('chat.composer.above-input', 'chat.quick-replies');

    // A Livewire component
    Outlet::livewire('chat.composer.above-input', 'chat-quick-replies');

    // A closure that returns a view, an Htmlable or a string
    Outlet::render('chat.composer.above-input', fn (array $data) => view('chat.badge', $data));

    // Alternative: hide only the package's default
    Outlet::view('chat.composer.above-input', 'chat.quick-replies', replace: true);
}
```

| Rule | What it means |
| --- | --- |
| Order | First the package's default, then everything registered, in the order it was registered. Several packages and the app may use the same outlet. |
| `replace: true` | Removes only the package's default, never what others registered. |
| Data | Every attribute on the tag except `name` and `key`. Kebab-case becomes camelCase: `:input-id` arrives as `$inputId` in a view, as `inputId` in a Livewire component and as `$data['inputId']` in a closure. |
| Livewire | A component only receives the data it declares as a `mount()` parameter or public property; extra objects are dropped and extra scalar values may become HTML attributes on the root element. |
| Loops | Inside a `@foreach` the package passes `:key="$row->id"`, so each Livewire component gets a stable `wire:key`. |
| Closures | A string you return is escaped. Return a view or an `Htmlable` for markup. |
| No wrapper | The tag adds no HTML element of its own; the package chooses the container. |

`Outlet::has('<name>')` tells you whether anything is plugged in. The name is always a literal string with at least three kebab-case parts, starting with the short package name (`chat.`, `entity-graph.`). `UiGuard` gives a hint for a name that breaks this rule, and `OutletsGuard` in `mortelos/dev-tools` fails the build for a view name absent from both the installed packages' manifests and your app's manifest. The full rules are in `COMPONENTS.md` in the package, under "Outlets".

## Outlets

An outlet is a place where you extend `mortelos/ui` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.5.0; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list ui
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| view | `ui.kitchen-sink.demo` | A demo place at the bottom of the kitchen sink at `/_mortel`, to show how a host fills an outlet. Default: a short text. | none |

## Boundaries

Use this package for reusable interface primitives. Keep customer branding, page composition, local copy and domain-specific workflows in the host app or in the package that owns that domain surface.
