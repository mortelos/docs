---
title: "MortelOS UI"
nav_title: "UI"
slug: "package-ui"
version: "0"
description: "The MortelOS component kit: x-mortel:: Blade components for host apps and packages, with its own shadcn-style implementation behind one switch."
section: "packages"
order: 53
status: "mvp"
audience: "developers"
package: "mortelos/ui"
canonical_path: "/docs/0/package-ui"
last_verified: "2026-10-19"
public: true
---

# MortelOS UI

`mortelos/ui` provides shared interface primitives for MortelOS host apps and reusable packages.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Components | Every screen of a host app or package is built from `<x-mortel::…>` Blade components: buttons, fields, selects, tables, menus, dialogs, toasts, the app frame, cards, tabs, key figures, page headers and action bars, lists, a command palette, steps and a status board. |
| Own kit | Its own implementation of those components, with the look of shadcn/ui, behind one switch. The same tags render through Livewire Flux until an app flips. |
| Theme | One set of semantic tokens (`--primary`, `--muted`, `--border`, `--radius`, …) that apps set to taste. |
| Guards | `UiGuard` keeps screens on the kit and Flux-free; `LivewireMonitor` budgets each interaction in browser tests. |
| View outlets | `<x-mortel::outlet>` marks a place in a package screen; `Mortelos\Ui\Outlet` lets an app plug its own view, Livewire component or closure into it. |

## Install

```bash
composer require mortelos/ui
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## The own kit

`mortelos/ui` has its own implementation of every component that host apps and packages use: Blade components with the look of shadcn/ui (through Basecoat, MIT), on Alpine and the browser's own `<dialog>` and popover, without Flux. Apps keep writing the same `<x-mortel::…>` tags. One switch per app picks the implementation:

```bash
MORTEL_UI_KIT=own   # the own kit; the default is flux
```

| Value | What the tags render |
| --- | --- |
| `flux` (default) | The Flux wrapper of each tag, as before. Updating `mortelos/ui` changes nothing while the switch is `flux`. |
| `own` | The own implementation of each tag that has one, otherwise its Flux wrapper. Compiled views go to a folder of their own, so flipping needs no `view:clear`. |

- The switch ships with `mortelos/ui` 0.5.8; every family is built from 0.5.13. Releases stay on 0.5.x; 0.6.0 removes Flux.
- Own components are at least as fast as the Flux wrappers they replace, in CPU time and in HTML bytes; the package's CI measures it for every component.
- The kitchen sink at `/_mortel` (environment `local`) shows every component; add `?kit=own` or `?kit=flux` to pick the mode.

## Documentation for AI agents

The package documents itself for an agent that builds a screen and must not invent anything. In an installed app it lives under `vendor/mortelos/ui/`. This set of files, `UiGuard::assertFluxFree()` and `UiGuard::assertAppComponents()` ship from 0.5.14.

| File | What it holds |
| --- | --- |
| `COMPONENTS.md` | The one entry point: the rules (kit first, no UI components of an app's own, ask the kit for a missing piece), a decision table from need to component, theming, the speed rules, the guards and every tag with its implementation. |
| `docs/components/<family>.md` | One page per family: the exact props and values, slots, events and hooks of each component, a minimal and a full example, and the mistakes to avoid. |
| `docs/recipes.md` | Copy-ready screens: a list page with filters, table, empty state and paging; a detail page with an inspector; a form with page actions; a settings page; a dashboard; a confirm dialog; a toast with undo; loading placeholders. |
| `docs/theme.md` | The theme tokens and how to set the brand. |
| `docs/migration.md` | How an app moves from Flux to the own kit. |

A doc-drift test in the package fails when a component or prop has no documentation, when an example or recipe does not render in both modes, when a recipe does not pass `UiGuard`, or when the decision table names a component that does not exist. The agent rules of [`mortelos/agent-standards`](/docs/0/package-agent-standards) ("TALL And UI") point agents to `COMPONENTS.md` first.

An app may have Blade components of its own for what is specific to it, on the kit and under the kit's rules: only `<x-mortel::…>` tags and HTML, no network calls, no server calls of their own, no scripts, no Flux, and field roles as props rather than business names. `UiGuard::assertAppComponents()` checks that.

## Moving an app to the own kit

`vendor/mortelos/ui/docs/migration.md` is the guide. In short:

1. **Step 0**, while the switch is still `flux`: `@mortelAppearance` and `@mortelScripts` in the layouts instead of `@fluxAppearance` and `@fluxScripts`; `kit.css` imported right after `flux.css` instead of `mortel.css`; `Flux::modal(…)` becomes `$this->dispatch('modal-show', name: …)` and `'modal-close'`; toasts use the `mortel-toast` event (in flux mode the kit hands it to Flux, without the undo action); `$flux.appearance` becomes `$store.mortel.appearance`; app CSS, JavaScript and tests that select `data-flux-*` hooks also select the kit hooks (`data-mortel-*`); app icons move to `resources/views/mortel/icon/`.
2. **The flip**, in one pull request: `MORTEL_UI_KIT=own`, the Flux selectors and `flux.css` out, the test suite green in own mode.
3. **Flux free**: `UiGuard::assertFluxFree()` fails on any Flux left in Blade, PHP, CSS and JavaScript.

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

With the own kit, import `kit.css` directly after Flux. Its tokens follow shadcn/ui's names (`--background`, `--primary`, `--muted`, `--border`, `--radius`, …); set them on `:root` and `.dark` after the import to change the look. An app that sets Flux's `--color-accent`, `--color-accent-content` and `--color-accent-foreground` keeps its brand: the kit reads them. The full list is in `vendor/mortelos/ui/docs/theme.md`.

```css
@import 'tailwindcss';
@import '../../vendor/livewire/flux/dist/flux.css';
@import '../../vendor/mortelos/ui/resources/css/kit.css';
```

The starter derives its brand from one token through the MortelOS theme layer `mortel.css`, between Flux and the kit. Without the own kit, set your brand colour in the host app's `resources/css/app.css` and import the theme layer directly after Flux (it ships with `mortelos/ui` 0.4.6 and later):

```css
@import '../../vendor/livewire/flux/dist/flux.css';
@import '../../vendor/mortelos/ui/resources/css/mortel.css';

@theme {
    --mortel-accent: #1466ff;
}
```

One token is enough. The layer derives readable button text and link tints for light and dark mode. Without any `--mortel-*` token Flux's own colours stay, with one exception: callouts (see below).

| Token | Used for | Default |
| --- | --- | --- |
| `--mortel-accent` | Primary buttons, active navigation, accent surfaces | Flux: zinc-800 in light, white in dark |
| `--mortel-accent-content` | Accent-coloured text in light mode | Derived: brand colour, darkened when needed to stay readable on white |
| `--mortel-accent-foreground` | Text on accent surfaces in light mode | Derived: white or black, whichever contrasts more |
| `--mortel-accent-dark` | Accent in dark mode | `--mortel-accent` |
| `--mortel-accent-content-dark` | Accent-coloured text in dark mode | Derived: brand colour, lightened when needed to stay readable on dark |
| `--mortel-accent-foreground-dark` | Text on accent surfaces in dark mode | Derived: white or black, whichever contrasts more |

- Set the tokens in `@theme` or on `:root`. On a deeper element they have no effect, because the derivation happens on `:root`.
- Import the theme layer after `flux.css`, never before it; before it, Flux's dark-mode rule wins and the brand colour disappears in dark mode.
- Do not set `--color-accent` yourself or add your own `.dark` rule for it; the layer handles both modes.
- Neutral greys follow Flux: redefine `--color-zinc-50` to `--color-zinc-950` in the same `@theme` block. White is not themeable; Flux and `mortelos/ui` use it directly for surfaces.
- A typo in a colour value makes the accent invalid (transparent buttons). Check a primary button in light and dark after changing it.
- The derivation uses CSS relative colour syntax and `round()`. It is tested in Chromium, WebKit and Firefox.
- Callouts in light mode: Flux colours their heading and text with shade 600 on shade 50, which stays below contrast 4.5 for twelve colours (green for `variant="success"` measured 3.08, yellow for `variant="warning"` 2.84). From `mortelos/ui` 0.5.1 the layer caps the lightness of that heading and text, so every Flux callout colour reaches at least 5.0. Colours that were already dark enough stay the same; the grey of a default callout gets slightly darker. Dark mode is unchanged. Measured in Chromium and WebKit.

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
