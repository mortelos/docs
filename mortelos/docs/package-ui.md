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
last_verified: "2026-09-28"
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

## Boundaries

Use this package for reusable interface primitives. Keep customer branding, page composition, local copy and domain-specific workflows in the host app or in the package that owns that domain surface.
