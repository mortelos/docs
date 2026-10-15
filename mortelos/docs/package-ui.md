---
title: "MortelOS UI"
nav_title: "UI"
slug: "package-ui"
version: "0"
description: "Shared UI primitives consumed by MortelOS packages and host apps."
section: "packages"
order: 53
status: "mvp"
audience: "developers"
package: "mortelos/ui"
canonical_path: "/docs/0/package-ui"
last_verified: "2026-10-14"
public: true
---

# MortelOS UI

`mortelos/ui` provides shared interface primitives for MortelOS host apps and reusable packages.

## Use It For

| Area | Package responsibility |
| --- | --- |
| UI primitives | Shared Flux-aligned Blade and Livewire interface building blocks. |
| Package surfaces | Reusable components that packages can consume without copying host app UI. |
| Design consistency | Common visual primitives for starter screens and package-owned views. |
| View outlets | `<x-mortel::outlet>` marks a place in a package screen; `Mortelos\Ui\Outlet` lets an app plug its own view, Livewire component or closure into it. |

## Install

```bash
composer require mortelos/ui
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Theme

Set your brand colour in the host app's `resources/css/app.css`. Import the MortelOS theme layer directly after Flux (it ships with `mortelos/ui` 0.4.6 and later):

```css
@import '../../vendor/livewire/flux/dist/flux.css';
@import '../../vendor/mortelos/ui/resources/css/mortel.css';

@theme {
    --mortel-accent: #1466ff;
}
```

One token is enough. The layer derives readable button text and link tints for light and dark mode. Without any `--mortel-*` token nothing changes: Flux's own colours stay.

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

`Outlet::has('<name>')` tells you whether anything is plugged in. The name is always a literal string with at least three kebab-case parts, starting with the short package name (`chat.`, `entity-graph.`). `UiGuard` gives a hint for a name that breaks this rule, and `OutletsGuard` in `mortelos/dev-tools` fails the build for a view name absent from both the installed packages' manifests and your app's manifest. The full rules are in `COMPONENTS.md` in the package, under `outlet`.

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
