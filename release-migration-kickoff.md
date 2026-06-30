# Kickoff: move apps onto package releases (dev-main → tagged `^x.y`)

Status: not started. Written 29 Jun 2026. Run this in a fresh, dedicated session.

## Goal

Today every app (`os`, `sijperda-os`, …) pins the `mortelos/*` packages to `dev-main`
(branch tips). That couples all apps to the latest package code: one change to e.g.
`framework` can silently break another app on its next `composer update`.

Target: **apps depend on tagged releases (`^0.5` etc.), not `dev-main`.** Each app then
upgrades a package deliberately, with version history and independent rollback. This is the
standing convention from now on (see "Convention" below).

## Why this is a project, not a quick task

- ~23 packages, cross-dependent, must be tagged **bottom-up** (a package's release may only
  require already-tagged versions of its deps).
- Existing tags **lag** the current `dev-main` code, so tagging is "capture current main as a
  release", not "the tags already exist".
- 6 os-dependencies have **zero tags** yet: `agent-standards`, `app-standards`,
  `channel-google-calendar`, `daily-planner`, `feedback`, `issue-factory`.
- It interleaves with the test-hardening initiative (per-package tests + CI).

## Current state (29 Jun 2026)

- Apps install from VCS at `dev-main` (branch tips). os `composer.json` requires are `dev-main`
  / `dev-codex/...` (entity-graph). Lock is git-based and green (see `os-composer-path-vs-vcs`
  memory). Local live-editing is via `composer link-local` (vendor symlinks), never path repos.
- Tagged already: framework v0.5.7, chat v0.3.8, ui v0.2.1, starter v0.5.0, most channels
  v0.2.0, dev-tools v0.1.3, mail/document-studio/policy-studio v0.1.0, entity-graph/overviews
  v0.1.1. (All behind current dev-main.)
- `branch-alias` exists on framework (dev-main→0.5.x-dev), ui (0.2.x-dev), chat (0.3.x-dev),
  and possibly others.
- A `mortelos/package-registry` repo exists but is empty — a private Composer registry is a
  possible direction. See also `os/docs/package-registry-access.md`.

## Phases

- **Phase 0 — dependency map (first deliverable).** Read every `mortelos/*` package's
  `composer.json`, build the require graph, derive the bottom-up tag order, and propose a
  version bump per package that captures current `dev-main`. Output a table; decide before
  tagging anything.
- **Phase 1 — tag bottom-up.** Per tier, leaves first: set each package's mortelos-dep
  constraints to the just-tagged versions, cut + push a release tag. First tags for the 6
  untagged packages.
- **Phase 2 — migrate the apps.** Change `os` (then `sijperda-os`) requires from `dev-main` to
  `^x.y`, `composer update`, run the full suite. Where a tag lacks something the app needs,
  bump that package. Keep the CI path-guard; live-edit still via `composer link-local`.
- **Phase 3 — borgen.** Document "how to cut a release", gate releases on per-package CI
  (test-hardening), and decide VCS-tags vs private registry.

## Open decisions (resolve in the session)

1. **Version bump per package** capturing current dev-main (patch vs minor; pre-1.0 semver).
2. **Sequencing vs test-hardening:** test a package green *then* tag, or tag now + harden
   later? This is the biggest call — it couples two large efforts.
3. **VCS tags vs private registry** (`package-registry` / Satis / private Packagist).
4. **Automation level:** manual tagging vs a release script/workflow (the issue-factory tooling
   could help).

## Paste-ready prompt for the new session

> I want to move our apps from depending on `mortelos/*` packages at `dev-main` (branch tips)
> to depending on **tagged releases** (`^x.y`). Context is captured in the memories
> `mortelos-release-model`, `os-composer-path-vs-vcs`, and `mortelos-test-hardening`, and in
> `~/Sites/mortelos/docs/release-migration-kickoff.md`. The packages live in `~/Sites/mortelos/*`
> and the main consuming app is `~/Sites/uteq/os` (also `~/Sites/uteq/sijperda-os`).
>
> Do NOT tag or change anything yet. Start with **Phase 0**: read every `mortelos/*` package's
> `composer.json`, build the inter-package dependency graph, and give me:
> 1. The bottom-up tagging order (tiers: which packages must be tagged before which).
> 2. For each package: its current latest tag, whether its current `dev-main` is ahead of that
>    tag, and a proposed next version (patch/minor, pre-1.0) that captures current `dev-main`.
> 3. Which packages currently require `dev-main`/`@dev` of another mortelos package (those
>    constraints must become `^x.y` at tag time) and which already use `^x.y`.
> 4. A flag on the 6 untagged os-deps (`agent-standards`, `app-standards`,
>    `channel-google-calendar`, `daily-planner`, `feedback`, `issue-factory`) needing a first tag.
> 5. The open decisions you need from me before Phase 1: version bumps, whether to gate tagging
>    on test-hardening (test-green-then-tag vs tag-now), and VCS-tags vs a private registry.
>
> Present Phase 0 as a table I can act on. Work read-only; do package and app git operations in
> isolated worktrees (I often have the same working trees checked out on other branches in
> parallel). After I approve the plan, we tag bottom-up, then migrate `os` (and `sijperda-os`)
> from `dev-main` to `^x.y` and prove the suite green.

## Guardrails to carry in

- Root cause first; releasing a broken package just hides the problem (see
  `feedback-root-cause-first`).
- Isolated git worktrees for all package/app changes; the user works in the same trees in
  parallel and HEAD moves.
- Keep apps installable from git the whole way (never reintroduce path-based locks).
