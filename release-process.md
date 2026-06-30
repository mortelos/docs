# Cutting a MortelOS package release

How to release a `mortelos/*` package and how apps consume it. This is the
standing model since the dev-main → tags migration (30 Jun 2026): **apps depend
on tagged releases (`^x.y`), never on branch tips.** One change to a package can
no longer silently break an app on its next `composer update`; each app upgrades a
package deliberately, with version history and independent rollback.

## The model in one paragraph

Each package is published as semver git tags. Apps (`os`, `sijperda-os`, …) require
caret ranges (`^0.5`, `^0.2`, …) resolved over VCS, and commit `composer.lock` so
installs are reproducible. Packages require each other with carets too. Local
live-editing is via `composer link-local` (vendor symlinks) — **never** path
repositories in `composer.json`.

## Dependency tiers (tag bottom-up)

A release may only require already-tagged versions of its deps, so tag in tier
order, leaves first. Within a tier, order is free.

- **Tier 0** (no mortelos deps): `framework` · `ui` · `app-standards` · `agent-standards` · `dev-tools`
- **Tier 1** (→ framework): `chat` · `mail` · `document-studio` · `daily-planner` · `feedback` · `issue-factory` · `channel-*` · `starter`
- **Tier 2** (→ chat [+ ui]): `overviews` · `policy-studio` · `widget-compliance` · `widget-document-feedback` · `entity-graph`

`framework` first deblocks everything (its `TenantTestCase` harness is imported by
downstream package tests). `chat` before the tier-2 packages that require `^0.3`.

## Cutting a release — procedure

Work in an **isolated git worktree** off `origin/main`, never in a checkout you
(or anyone) might have on another branch. `git tag`/`log` are read-only and safe.

1. **Worktree**: `git worktree add /tmp/wt-<pkg> origin/main`.
2. **Green gate** (this is the release gate — test-green-then-tag):
   `composer install && composer ci` (PHPStan level max + Pest/PHPUnit). Must be
   fully green. Private tier-1/2 packages install standalone because their
   `composer.json` lists VCS repos for `framework`/`chat`; auth comes from
   `~/.config/composer/auth.json` (github-oauth).
3. **Tighten constraints**: any `@dev` / `dev-main` constraint on another mortelos
   package becomes a caret on its current tag (`framework @dev → ^0.5`). Packages
   already on carets need no change.
4. **Tag annotated + push**:
   `git tag -a v0.3.9 origin/main -m "v0.3.9 — <what>; phpstan clean, N tests green"`
   then `git push origin v0.3.9`. If main needed a commit (constraint rewrite,
   reconcile), push the branch to `main` first (fast-forward).
5. **Clean up**: `git worktree remove …`.

A package whose `origin/main` is already green and ahead of its last tag just needs
steps 2 + 4 (no merge). The latest tag stays the source of truth.

## Reconciling divergent branches

If a tag or a feature/test-hardening branch has diverged from `main` (the tag was
cut on a release branch, or a feature lives only on `test-hardening`/a codex
branch), converge **onto main** before tagging — do not tag a parallel line:

- Bring the work onto a main-based worktree (`git merge`, `git checkout <branch> --
  <files>`, or a `git apply --3way` patch). Resolve conflicts toward the superset
  that the apps actually run.
- Make it green (fix what the divergence carried — e.g. a feature that was never
  PHPStan-clean), then tag from `main`.
- Worked example: `entity-graph v0.2.0` merged the codex branch (the audit feature
  the apps ran) with `test-hardening` (the Pest suite); `framework v0.5.8` salvaged
  the inbox-action-payload feature off `test-hardening` and fixed its findings.

## Version bumps (pre-1.0)

Default to a **patch** bump that captures current `main` (additive work, tests,
fixes). Reason: it stays within dependents' existing caret range (`chat ^0.3`,
`ui ^0.2`), so no cascade of constraint bumps. Use a **minor** only for a genuine
behavioural/API change — and remember a pre-1.0 minor (`0.2.x → 0.3.0`) falls
*outside* `^0.2`, so every dependent's constraint must move too.

## Consuming a release in an app

- App `composer.json` requires carets (`mortelos/framework: ^0.5`), with a VCS repo
  per mortelos dep. **No `path` repositories** — the `../*` / `../framework`
  wildcard path-lock is the recurring CI/deploy breaker; remove on sight.
- Upgrade deliberately: `composer update mortelos/<pkg>` (without
  `--with-all-dependencies` to keep the bump focused), run the app suite, commit
  the lock. Land via PR to the app's `main`.
- Live-edit a package against an app via `composer link-local` (vendor symlinks),
  which is local and uncommitted — never path repos in the committed manifest.

## Guardrails

- **Root cause first.** If the green gate is red, fix the cause; never tag a broken
  package or weaken the gate (no PHPStan baseline / `ignoreErrors` / `@phpstan-ignore`
  to force green).
- **Isolated worktrees** for every package/app change; live checkouts and their
  HEADs stay untouched.
- **Keep apps installable from git** the whole way; never reintroduce path-based
  locks.
