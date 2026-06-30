# Handoff-prompt — test-hardening afmaken (clean session)

> Plak alles onder de streep in een verse Claude Code-sessie (cwd `~/Sites/mortelos`).

---

Je maakt het **mortelos package test-hardening-traject** af. De volledige context staat in
`~/Sites/mortelos/docs/test-hardening.md` (bron van waarheid) en in je projectgeheugen
(`memory/mortelos-test-hardening.md`). **Lees die twee eerst.** Ultracode mag aan voor de
authoring-golven.

## Wat al klaar is
- Alle 22 mortelos-packages hebben eigen tests (~560), 6 channels hebben slice-2 boundary-tests
  (75) op de gedeelde `Mortel\Testing\TenantTestCase`-harnas. Alles `composer test`-groen.
- Getagd: **framework v0.5.5** (harnas), **dev-tools v0.1.3** (gate-commando).
- PR's open: 22 slice-1 (test-hardening branch), 6 slice-2 boundary (#4, basis=test-hardening),
  os Fase 0b (#24), 2 bug-fixes (channel-google-drive#5, app-standards#2).
- framework harnas-src is al phpstan-schoon (commit 3e83fcf op framework `test-hardening`).

## Werkomgeving (belangrijk)
- Elke package onder `~/Sites/mortelos/<naam>` is een eigen git-repo. Apps: `~/Sites/uteq/os`
  (productie, staat op feature-branch met veel WIP — NIET aanraken; gebruik een git-worktree
  vanaf `origin/main` voor os-PR's) en `~/Sites/uteq/sijperda-os`.
- `composer`/`pest` hebben netwerk nodig → roep Bash aan met `dangerouslyDisableSandbox: true`.
- Standalone install per package: `composer update --working-dir=<pkg> --no-interaction --no-progress`
  (VCS naar private `mortelos/framework`; Composer `auth.json` staat in `~/.config/composer`).
- Git-identity is leeg → commit inline:
  `git -c user.name="Nathan Jansen" -c user.email="info@uteq.nl" commit ...`
  Commit-footer: `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`.
- Werkwijze die werkt: een authoring-workflow schrijft testbestanden (geen netwerk) → daarna
  draai jij in de main-loop per package `composer update` + `composer test`/`analyse` + fix.

## Te doen (prioriteitsvolgorde)

### 1. phpstan / `composer ci` schoonmaken (HOOG — de ci-gate = [@analyse,@test])
De tests slagen overal, maar `composer analyse` (phpstan, larastan level max) faalt op
TEST-bestanden in 5 packages. Per package: `cd <pkg> && vendor/bin/phpstan analyse --no-progress
--error-format=raw --memory-limit=2G` om de lijst te krijgen, fix, dan `composer ci` groen.
- **framework** ~21 test-errors (`tests/Unit/Agent/DefaultAgentModelGatewayTest.php` 15,
  `tests/Feature/Spike/SpikeServiceProviderTest.php` 4, `tests/Feature/Agent/AgentApprovalServiceTest.php` 2).
  Plus ~16 pre-existing (waarvan user-WIP `InboxDetailToolTest.php` 9 — die zit NIET in de v0.5.5-tag,
  raak 'm niet aan). De src/Testing-harnas is al schoon (3e83fcf).
- **dev-tools** 8, **channel-telegram** 4, **app-standards** 4, **chat** 2. **feedback** heeft geen
  phpstan/larastan (analyse-stap n.v.t.).
- Foutpatronen + fixes: `method.alreadyNarrowedType` (redundante `assertInstanceOf` → weghalen);
  `method.notFound`/`method.nonObject` op mocks (typ de mock-variabele, of assert via een methode
  die op het echte type bestaat); `offsetAccess.notFound` op `array|null` (guard met `is_array`);
  `argument.type` op `::create()` (voeg `@property`-hints toe aan het model — zie hoe het in
  `framework/src/Testing/Models/TestTenant.php` is gedaan).
- Commit op de `test-hardening`-branch van elke package (werkt de bestaande slice-1-PR bij), of een
  `fix/phpstan`-branch. GEEN `@phpstan-ignore`/baseline gebruiken — fix de oorzaak.

### 2. framework v0.5.6 re-taggen (na #1)
v0.5.5 is getagd zonder analyse. Zodra framework's `composer ci` groen is: schone tag via
`git checkout -b release/v0.5.6 origin/main` → cherry-pick ALLEEN je eigen commits (golf-D tests +
harnas + phpstan-fixes, NIET user-WIP `176c7e7`) → `composer ci` groen verifiëren →
`git -c user.name=... tag -a v0.5.6 -m ...` → `git push origin v0.5.6`. Daarna in de channels
`composer update mortelos/framework`.

### 3. Event-sourcing-harnas (slice 3) — unlockt de gedeferde boundary-tests
Commands die een Channel via de event-sourced `ChannelAggregate` AANMAKEN (google-drive
`mortel:channel:drive:setup`, plaud `uteqos:channel:connect-plaud`) hadden de projector-keten nodig,
die nog niet synchroon in de harnas draait. Stappen:
- Voeg `Spatie\EventSourcing\EventSourcingServiceProvider::class` toe in
  `Mortel\Testing\TenantTestCase::getPackageProviders` (config `snapshot_repository`; framework
  levert de `events`/`snapshots`-migraties al). Voor google-drive staat dit al per-channel in
  `channel-google-drive/tests/Support/TenantTestCase.php` — til het op naar de gedeelde harnas.
- Zorg dat de Mortel-projectors synchroon projecteren (kijk hoe framework's eigen aggregate→projectie
  tests dit doen). Verifieer met een smoke-test: maak een channel via een aggregate-pad → de
  `channels`-rij verschijnt.
- Her-activeer dan de gedeferde asserts: `channel-google-drive` `DriveSetupCommandTest` (channel-count
  na create) en `channel-plaud` `PlaudConnectCommandTest` (de 3 create-via-aggregate tests die nu
  ontbreken). Deze zitten op de `slice-2-boundary`-branches.
- Dit is een framework-wijziging → vergt v0.5.6 (zie #2) of een per-channel stopgap.

### 4. Overige slice 2 (per package, authoring-workflow + verify)
Gebruik telegram's slice-2 (`channel-telegram` branch `slice-2-boundary`,
`tests/Feature/Boundary/*` + `tests/Support/TenantTestCase.php`) als template-patroon.
- **entity-graph**: Actions (BuildEntityGraph/SearchEntityGraph/FindEntityGraphPath) + HTTP
  controllers/resources + ServiceProvider-wiring (DB + access-context; `TenantTestCase`).
- **policy-studio**: PolicyProposalService/PolicyConditionProposalService/AnswerPolicyChangeRequest +
  Livewire governance-componenten.
- **document-studio**: volledige sign-off-lifecycle (DocumentAggregate, register/collectie-actions) —
  event-sourcing (zie #3).
- **mail**: pgsql tsvector-zoektak (vergt pgsql; anders expliciet skippen/markeren).
- **widget-compliance / widget-document-feedback / ui / daily-planner**: Livewire-component-render.
- **chat**: pdf/xlsx attachment-extractie (externe binaries pdftotext/openspout — mock of skip).
Markeer wat event-sourcing/externe-binaries vereist expliciet als latere slice i.p.v. rode tests.

### 5. Merges (na review; jij/de gebruiker beslist, maar bereid voor)
- 6 boundary-PR's (#4 per channel) stacken op test-hardening → mergen ná de slice-1-PR's.
- os Fase 0b (#24): éérst `composer update mortelos/dev-tools` in os (pakt v0.1.3), dan mergen.
- Bug-fix-PR's: channel-google-drive#5, app-standards#2.

### 6. policy-studio "zet X uit"-deny (beslissing)
De intent-resolver detecteert "zet \<X\> uit" (gesplitst) niet als deny (alleen aaneengesloten
'zet uit'/'uitzetten'). Dit is een debatabele product-design-keuze, geen duidelijke bug. Leg voor
of behandel als kleine verbetering (regex voor "zet ... uit") mét test.

## Tracking bijwerken
Houd `docs/test-hardening.md` + het projectgeheugen bij. Linear: initiative
"Package test-hardening (mortelos)", project "Test-hardening packages", issues UTEQ-586..608
(via de claude.ai Linear MCP-tools). Sync state + comments per afgeronde stap.

## Belangrijke principes
- Verifieer `composer ci` (analyse + test), niet alleen `composer test`.
- Liever een kleinere groen-bare set dan rode tests; markeer event-sourcing/externe-binary-paden als
  latere slice.
- Class-based tests (`extends ...TenantTestCase`) i.p.v. Pest-functioneel om `uses()->in()`-conflicten
  te vermijden.
- Raak os' productie-working-tree niet aan; gebruik een git-worktree vanaf `origin/main`.
