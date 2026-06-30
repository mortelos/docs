# Package test-hardening — mortelos

> **Bron van waarheid** voor het degelijker maken van de mortelos-packages met eigen tests.
> Gestart 2026-06-18. Eigenaar: Uteq. Tracking spiegelt naar Linear-initiative *Package test-hardening*.

## Waarom

22 packages in `~/Sites/mortelos`. Nulmeting (2026-06-18): **6 met eigen tests, 16 zonder, 15 high-risk**.
De ontbrekende-tests-packages worden vandaag alleen *indirect* getest door de consumerende apps
(vooral `uteq/os`, 458 tests). Kern-probleem:

> **Er bestaat geen gate die afdwingt dat een package z'n eigen tests draait.** `os` draait in
> `quality-gates` alleen de app-suite; `package-governance` checkt enkel beslissings-administratie.
> 17 packages hebben alleen een `phpstan.yml` (analyse, geen tests); slechts `dev-tools` en `site`
> draaien een testsuite in eigen CI.

Gevolg: geen ownership per package, `sijperda-os` is grotendeels blind, refactoren is eng, en
trage feedback (een driver-bug vereist de hele os-suite + postgres).

## De norm

- **Pest 4 bovenop `orchestra/testbench`** (Pest draait op PHPUnit → geen runtime-verlies).
- **Gedeelde basis:** `Mortel\Testing\OsTestCase` (geleverd vanuit `framework/src/Testing`). Elke
  package krijgt een dunne `tests/Support/TestCase.php` die `OsTestCase` extend (of direct
  `Orchestra\Testbench\TestCase` als er geen framework-providers nodig zijn), met eigen
  `getPackageProviders()` + migraties. DB = sqlite `:memory:`.
- **Skelet per package:** `composer.json` (require-dev + `autoload-dev` + scripts), `phpunit.xml`
  (Unit+Feature suites), `tests/Pest.php`, `tests/Unit/` + `tests/Feature/`, gedeelde
  `.github/workflows/ci.yml`, `phpstan.neon`.
- **Scripts:** `test` → `vendor/bin/pest`, `test:unit`, `test:feature`, `analyse`, `format`,
  `ci` = `[@format --test, @analyse, @test]`.
- framework hoeft niet in één keer naar Pest — Pest draait z'n PHPUnit-classes ongewijzigd mee.

## De gate (Fase 0 — de ruggengraat)

1. Herbruikbaar skelet-sjabloon genereren uit de norm.
2. Nieuw artisan-commando `mortelos:package-tests:check` in **dev-tools**: itereert over
   geïnstalleerde `mortelos/*` packages en eist een (groen) `test`-script.
3. Aanhaken in `os/composer.json` als `test:packages`, in `quality-gates`, en een assertie in
   `MortelPackageWiringTest` ("elke require'de mortelos/*-package heeft een test-script").
4. `package-first-governance` skill bijwerken met de regel.
5. Per-package `ci.yml` standaardiseren (de 17 phpstan-only repos draaien voortaan ook `composer test`).

**Veiligheidsprincipe:** eerst tests *naar binnen toevoegen*, app-tests in `os` laten staan,
duplicaten pas in een aparte latere pass snoeien → dekking kan nooit tussentijds dalen.

## Status

Legenda: ⬜ te doen · 🔨 bezig · ✅ klaar · 🟡 eigen tests met gaten

| Package | Golf | Risico | Eigen tests | Indirect (os) | Effort | Status | Eerste/grootste gat |
|---|---|---|---|---|---|---|---|
| channel-telegram | A | 🔴 | 23 ✅ | good | M | 🟢 kern | pilot groen (23 tests, 84 asserts); boundary-tests = slice 2 |
| channel-gmail | A | 🔴 | 23 ✅ | good | M | 🟢 kern | slice 1 groen; jobs/oauth/classify = slice 2 |
| channel-google-drive | A | 🔴 | 22 ✅ | good | M | 🟢 kern | slice 1 groen; push-job/oauth/commands = slice 2 |
| channel-moneybird | A | 🔴 | 19 ✅ | good | M | 🟢 kern | slice 1 groen; sync-actions/commands = slice 2 |
| channel-fireflies | A | 🔴 | 17 ✅ | good | M | 🟢 kern | slice 1 groen; webhook/job/commands = slice 2 |
| channel-plaud | A | 🔴 | 24 ✅ | partial | M | 🟢 kern | slice 1 groen; poll-job/store = slice 2 |
| entity-graph | B | 🔴 | 43 ✅ | partial | L | 🟢 kern | Matcher/Formatter/Presenter/Metrics units; Actions/HTTP = slice 2 |
| policy-studio | B | 🔴 | 59 ✅ | good | L | 🟢 kern | intent-resolver heuristiek + agent-schema; proposal/Livewire = slice 2 |
| chat | B | 🔴 | 63 ✅ | partial | M | 🟢 kern | WidgetRegistry/Renderer + attachment-extractor + middleware |
| overviews | B | 🔴 | 19 ✅ | good | M | 🟢 kern | QueryPlanner + save-prompt-widget; Saver/datasource = slice 2 |
| document-studio | B | 🔴 | 17 ✅ | partial | M | 🟢 kern | sign-off finalize/listener-branches; volledige lifecycle = slice 2 |
| mail | B | 🔴 | 31 ✅ | partial | M | 🟢 kern | upsert/search(sqlite)/embedding-job; pgsql-tak = slice 2 |
| agent-standards | C | 🟢 | 6 ✅ | none | S | ✅ | publish-registratie + resource-integriteit |
| ui | C | 🟢 | 18 ✅ | thin | S | ✅ | view-namespace + render-smoke + publish-contract |
| widget-compliance | C | 🟢 | 9 ✅ | partial | M | 🟢 kern | definition + registry-registratie; Livewire-render = slice 2 |
| widget-document-feedback | C | 🟢 | 8 ✅ | partial | M | 🟢 kern | definition + registry-registratie; annotator-render = slice 2 |
| framework | D | 🔴 | +15 ✅ | partial | M | 🟢 gap | gateway/approval/spike groen; pre-existing `InboxDetailToolTest` faalt (niet van mij) |
| feedback | D | 🔴 | +13 ✅ | partial | M | 🟢 gap | GitHub/Linear-clients groen; pre-existing `ProcessFeedbackReportJobTest` faalt standalone |
| issue-factory | D | 🔴 | 47 ✅ | partial | M | ✅ | `LocalMarkdownIssueSource` + Dossier-markdown gedekt |
| daily-planner | D | 🟡 | 29 ✅ | good | M | ✅ | Actions + Models in-package (Testbench + migraties) |
| app-standards | D | 🟡 | 18 ✅ | none | M | ✅ | disabled/ongedekte config-takken gedekt |
| dev-tools | D | 🟡 | 38 ✅ | none | M | ✅ | merger/discovery/decision-log + gate-command |

## Golven (uitvoervolgorde)

- **Golf A — externe-API channels** (hoogste risico): telegram (pilot) → gmail, google-drive, moneybird, fireflies, plaud.
- **Golf B — kernlogica:** entity-graph, policy-studio, chat, overviews, document-studio, mail.
- **Golf C — low-risk presentatie/standaarden:** agent-standards, ui, widget-compliance, widget-document-feedback.
- **Golf D — gaten dichten in bestaande suites:** framework, feedback, issue-factory, daily-planner, app-standards, dev-tools.
- **Fase 4 (apart, na groen):** verhuisde unit-tests uit `os` snoeien, integratie/wiring behouden.

## Werkwijze per package (bewezen in de pilot)

`skelet plaatsen` → `move-inward` (unit-achtige os-tests porten) → `nieuwe tests` (de leemtes
hierboven) → `composer test` groen → **adversariële verify** (sceptische agents: toetsen de tests
echt gedrag? geen `assertTrue(true)`, mocks niet over-gespecificeerd).

De volledige per-package analyse (move-inward-kandidaten, keep-in-app, aanbevolen nieuwe tests)
staat in de inventarisatie-output van workflow `mortelos-test-hardening-grondslag`.

## Voortgang (log)

**2026-06-18**
- Tracking opgezet: dit doc + Linear-initiative *Package test-hardening (mortelos)* + project *Test-hardening packages* (UTEQ-586 t/m UTEQ-608, 1 issue per package).
- **Pilot `channel-telegram` — kern groen.** Skelet (Pest 4 + `OsTestCase` + sqlite `:memory:`) + composer-scripts toegevoegd; unit-tests + registry-wiring + delivery-chunking geport uit os; nieuwe `TelegramApiClientTest` (de leemte) toegevoegd. `composer test`: **23 passed, 84 assertions, 0 notices**.
- Adversariële verify (2 lenzen) uitgevoerd en verwerkt: caption-mapping via unit-mock i.p.v. multipart-introspectie, `hasFile`-assertions op de multipart-velden, response-edge-cases (ontbrekend `document`/`photo`-veld), chunk ≤ 4096-assertie, en de `getimagesize`-notice weggewerkt.

**Sjabloon-lessen voor de golven**
1. Assert de request-VORM, niet alleen call-counts.
2. Multipart form-velden zijn niet leesbaar via `$request['x']` → toets die op de unit-laag met een gemockte client.
3. Overweeg een gedeelde `tests/Support/HttpAssertions.php` + fixture-factory vóór brede uitrol over de channels.
4. De echte risico's per channel zitten in webhook-controller / inbound-job / setup-health-commands — plan die als aparte slice per channel.

- **Fase 0a — gate-command gebouwd & groen.** `mortelos:package-tests:check` toegevoegd aan **dev-tools** (warn-only; `--strict` voor blocking; scant `mortelos/*`+`uteq/*` requires op een `test`-script), geregistreerd in de ServiceProvider, met 5 nieuwe tests. dev-tools-suite: **19 tests, 78 assertions, OK**. Gate-regel toegevoegd aan de `package-first-governance` skill.

- **Golf A — alle 6 channels slice-1 groen.** Via een authoring-workflow (5 agents schrijven parallel) + verify per channel: telegram 23, gmail 23, google-drive 22, moneybird 19, fireflies 17, plaud 24 → **122 tests groen**. Elk: skelet + driver-unittests + `ApiClient` Http::fake-tests (de leemte) + provider-registratie. Eén fragiele Content-Type-assertie in google-drive verstevigd naar een body-check.

**Bevinding (→ slice 2):** `GoogleDriveApiClient::uploadContents` zet `Content-Type: multipart/related` via `withHeaders`, maar `->withBody($body)` kan die overschrijven naar `application/json` — echte Google-uploads zouden kunnen falen. Verifiëren + evt. fixen in google-drive slice 2.

- **Golf C — 4 low-risk packages groen.** agent-standards 6, ui 18, widget-compliance 9, widget-document-feedback 8 → **41 tests**. Geleerde patronen: (1) packages zónder framework-dep (agent-standards, ui) extenden direct `Testbench`, niet `OsTestCase`; (2) chat-widgets hebben alleen de `WidgetRegistry`-binding nodig — boot NIET de volledige `UteqChatServiceProvider` (registreert host-routes die in Testbench falen), maar bind de registry scoped + voeg `LivewireServiceProvider` toe; (3) Flux-form-componenten vereisen een gedeelde lege `ViewErrorBag` bij `Blade::render`; (4) `ServiceProvider::publishableGroups()` geeft een lijst (gebruik `toContain`, niet `toHaveKey`).

- **Golf B — alle 6 zware packages slice-1 groen.** entity-graph 43, policy-studio 59, chat 63, overviews 19, document-studio 17, mail 31 → **232 tests**. Fixes/patronen voor unit-only/zware packages: (1) ontbrekende `tests/Feature`-map → `.gitkeep` (phpunit-suite-dir moet bestaan); (2) `#[DataProvider]`-attribuut i.p.v. doc-comment `@dataProvider` (PHPUnit 12); (3) een `final` action is niet te mocken → generieke `Mockery::mock()` onder de container-key (de listener resolvet duck-typed via `app()`); (4) tests afgestemd op het échte gedrag van de code.

**→ Mijlpaal: alle 16 eerder ongetestte packages hebben nu een groene `composer test` (Golf A 6 + C 4 + B 6).**

**Bevinding (→ slice 2):** `PolicyChangeIntentResolver` detecteert "zet \<X\> uit" (gesplitst) niet als deny-intent (alleen aaneengesloten 'zet uit'/'uitzetten') — kleine resolver-gap.

- **Golf D — gaten gedicht in de 6 packages die al tests hadden.** framework +15 (DefaultAgentModelGateway/AgentApprovalService/Spike), feedback +13 (GitHub/Linear-clients), issue-factory 47 (LocalMarkdownIssueSource + Dossier), daily-planner 29 (Actions+Models in-package), app-standards 18 (disabled-takken), dev-tools 38 (merger/discovery/decision-log). Al mijn nieuwe tests groen. Fixes: `realpath()` voor macOS `/var`→`/private/var`; `Date::useDefault()` in setUp tegen facade-pollution.

- **Alle 17 + 6 = repo's gecommit op branch `test-hardening` (geen push).** Auteur Nathan Jansen <info@uteq.nl>, inline (geen globale git-config). framework chirurgisch: alleen de 3 eigen testbestanden, pre-existing WIP (`InboxDetailToolAdapter`/`InboxDetailToolTest`) ongemoeid.

**Pre-existing falers (NIET door dit traject veroorzaakt, te onderzoeken):**
- framework `tests/Feature/Agent/InboxDetailToolTest.php` — 2 errors (InboxItemFactory guarded-attributes); untracked WIP.
- feedback `tests/Feature/ProcessFeedbackReportJobTest.php` — 5 failures in standalone-modus (gecommit; mogelijk host-app-context nodig).

**Bevinding (→ slice 2):** app-standards provider gate't `passwords` via `booleanConfig('passwords')` (de hele array → altijd truthy), niet via `passwords.enabled` zoals de `.enabled`-conventie elders — inconsistentie; disablen kan alleen door `passwords` zelf op `false` te zetten.

- **Alle 22 packages gecommit + gepusht op branch `test-hardening`, 22 PR's geopend** (auteur Nathan Jansen <info@uteq.nl>; entity-graph/overviews op hun feature-branch als basis). PR-links gekoppeld aan de Linear-issues UTEQ-586…608.

- **Fase 0b — PR klaargezet:** [uteq/os#24](https://github.com/uteq/os/pull/24) voegt een warn-only `test:packages`-stap toe aan `composer quality-gates` (draait `mortelos:package-tests:check`, degradeert netjes vóór de dev-tools-release). Gemaakt via een git-worktree vanaf `origin/main` zodat os' WIP-working-tree ongemoeid bleef. Merge ná de dev-tools-release (PR mortelos/dev-tools#2) + `composer update`. De gate-regel staat al in de `package-first-governance` skill op main.

- **Slice 2 — gedeelde tenancy-harnas gebouwd & groen.** `Mortel\Testing\TenantTestCase` (+ `FakeDatabaseBootstrapper`, `FakeTenantDatabaseManager`, `TestTenant`/`TestUser`-modellen) toegevoegd aan **framework/src/Testing** — brengt voor het eerst echte stancl-tenancy op de gedeelde sqlite `:memory:` in framework's testbench (één PDO via de fake bootstrapper). Bewezen door `TenantHarnessSmokeTest` (3 tests: tenant-by-slug, channel-persist onder tenancy, owner-via-pivot). framework-suite: 251 tests, geen regressie. Gecommit + gepusht op framework `test-hardening` (PR #3 bijgewerkt).

- **Releases getagd:** **framework v0.5.5** (harnas + golf-D tests; schoon via cherry-pick van `origin/main`, ZONDER de InboxDetailTool-WIP → 249 tests groen) en **dev-tools v0.1.3** (gate-command). Beide live op de remotes.

- **Slice 2 — boundary-laag uitgerold over alle 6 channels.** Na framework v0.5.5 in elke channel-vendor: telegram 14, gmail 11, google-drive 16, moneybird 14, fireflies 16, plaud 4 → **75 boundary-tests groen** op de `TenantTestCase`-harnas (webhook-secret/tenant-routing, OAuth-callbacks, inbound→InboxItem, health/setup-commands). Gecommit op `slice-2-boundary`-branches per channel; **PR #4 in elke channel-repo** (basis = test-hardening). Fixes onderweg: redirect-naar-root-assertie (`http://localhost`), Spatie `EventSourcingServiceProvider` toevoegen aan de channel-base (config `snapshot_repository`), plaud Pest-binding → class-based.

**Bevinding (→ event-sourcing-slice):** commands die een Channel via de `ChannelAggregate` *aanmaken* (google-drive setup, plaud connect) hebben de volledige event-sourcing **projector-keten** nodig — die draait nog niet synchroon in de harnas. Die channel-projectie-asserts zijn gedefer. Volgende harnas-stap: Spatie-provider + projectionist in `Mortel\Testing\TenantTestCase` zelf (framework v0.5.6) zodat álle channels dit erven.

**2026-06-26**

- **phpstan / `composer ci` schoongemaakt in alle 5 packages.** De `ci`-gate = [@analyse,@test]; analyse (larastan level max) faalde op test-/src-bestanden. Root-cause-fixes (GEEN `@phpstan-ignore`/baseline/silencing-cast):
  - **framework** (13 non-WIP fouten): `phpstan/phpstan-mockery` (dev) toegevoegd → typeert `Mockery::mock(Class::class)` als intersection (lost 14 `DefaultAgentModelGatewayTest`-fouten op); redundante `assertInstanceOf` weg (Spike/gateway) → i.p.v. `app()->bound()` + `Scope`-contract via `class_implements`; enum-case-lijsten via `json_encode(...)` (string|false, niet door phpstan te vouwen → set/volgorde/aantal-garantie blijft); `AgentRunStep::output` (array|null) genarrowd met `assertIsArray`; `app()`-helper i.p.v. nullable `$this->app`. De InboxDetailTool-WIP (176c7e7) bleef onaangeroerd (10 WIP-fouten resteren op test-hardening, vallen buiten de release).
  - **dev-tools** (8): iterator-items getypeerd als `SplFileInfo`; tautologische `assertArrayHasKey` → value-asserts.
  - **channel-telegram** (4, src): altijd-ware `is_array`-guard weg + keys via `stringKeyed`; `chunk()` `$limit` als `int<1,max>`; redundante `?? null` op `getimagesize()[2]` weg. + Pint-formatting (import-ordering) van de slice-1-testfiles.
  - **app-standards** (4): `$this->app` genarrowd via `assertNotNull`; mutable-date-assert via runtime-mutatie; Http-facade-union genarrowd naar `Response`.
  - **chat** (2, src): `UploadedFile::extension()` (string|null) → `?? ''` vóór `strtolower`; `array_values()` op migratie-paden zodat `array_splice`-offset een int is.
  - Authoring via workflow (4 packages parallel) + adversariële verificatie per package (phpstan 0 + pest groen + gedrag behouden, alle 4 **pass**); framework zelf in de main-loop (WIP-grens). Alle 5 gecommit + gepusht op `test-hardening` → de 4 package-PR's worden analyse-CI-groen.
- **framework v0.5.6 getagd + gepusht.** Schone branch `release/v0.5.6` vanaf origin/main → cherry-pick van alleen mijn commits (golf-D tests + harnas + harnas-phpstan-clean + test-phpstan-clean), ZONDER de WIP `176c7e7`. **`composer ci` groen: PHPStan 0 errors, PHPUnit 249 passed.** Tag gepusht naar `mortelos/framework`.

- **Event-sourcing-harnas (slice 3) KLAAR → framework v0.5.7 getagd + gepusht.** `Mortel\Testing\TenantTestCase` registreert nu Spatie's `EventSourcingServiceProvider` + de Mortel-projectors (`config('event-sourcing.projectors')`, overridebaar via `eventSourcingProjectors()`, `queue=null`) zodat commands die een Channel via de `ChannelAggregate` aanmaken **synchroon projecteren** in de `channels`-tabel — geen handmatige replay. Bewezen door een nieuwe `TenantHarnessSmokeTest`-case (CreateChannel → channels-rij). De per-channel ES-stopgap (google-drive) is opgetild naar de gedeelde harnas. Ontbrekende stuk t.o.v. v0.5.6 was de projector-registratie (google-drive had wél de provider maar géén `projectors`-config → projectie liep niet). v0.5.7 `composer ci` groen: PHPStan 0, PHPUnit 250. Channels-rollout: na `composer update mortelos/framework` → v0.5.7 op alle 6 channels groen geverifieerd (google-drive 16, plaud 5, telegram 14, gmail 11, fireflies 16, moneybird 14). **Gedeferde tests gere-activeerd:** channel-google-drive `DriveSetupCommandTest` (channels-rij na drive:setup) + channel-plaud `PlaudConnectCommandTest` (3 happy-path create-via-aggregate tests: connected channel, config-credential-fallback, encrypted credentials) — gecommit + gepusht op de `slice-2-boundary`-branches.

- **Overige slice-2 (#4) — 5 hoogwaardige packages groen.** Discovery-workflow (9 packages) bracht groen-baar vs. defer in kaart; daarna een author+verify-workflow (auteur + adversariële reviewer per package, alle 5 **pass**, suites groen, meaningful). Toegevoegd: **document-studio** (sign-off-lifecycle CRUD + DocumentAggregate sign-off-pad — `tests/Support/TenantTestCase` overridet `eventSourcingProjectors()` met `DocumentProjector`, bewijst dat de v0.5.7 ES-harnas generaliseert voorbij channels), **entity-graph** (Build/Search/FindPath-actions tegen echte Entity/EntityLink-tabellen + HTTP-controllers + widget-manifest/tool-metadata-contract; 66 groen), **mail** (Project Mail Received/Sent-listeners, upsert, sqlite-search, embedding-job met Embeddings::fake), **chat** (DOCX/CSV-extractie + attachment-formatter, in-test fixtures, geen externe binaries), **widget-compliance** (compliance-intake Livewire-render + visibility-mock + events). Twee reviewer-mustFix opgelost: chat temp-file-leak (tempnam-wees → direct naar tempnam-pad) en entity-graph 2 ten onrechte gedeferde contract-tests (widget `definition()` + tool-metadata) toegevoegd. Gecommit + gepusht op `test-hardening`. **Expliciet gedeferd:** mail pgsql-tsvector (vergt postgres), chat pdf/xlsx (pdftotext/openspout externe binaries), diverse Livewire-render/agent-execute/OAuth-paden. **Al goed gedekt (lager geprioriteerd, niet geauteerd):** policy-studio-services, widget-document-feedback, ui, daily-planner (slice-1 dekt de kern).

- **Merge-readiness (#5) + CI-BLOKKADE gevonden.** De per-package GitHub-Actions `phpstan.yml` faalt op `test-hardening` — **niet** op phpstan maar bij **Install dependencies**: `composer install` → *"Could not authenticate against github.com"* (exit 1). Geverifieerd op dev-tools (run draaide op exact mijn commit 6d7f945) én chat. Oorzaak = de runner-secret `MORTELOS_CI_TOKEN` is verlopen/leeg/rate-limited bij dist-downloads (bv. symfony/error-handler). **Dit is infra, niet mijn code** — lokaal is alles phpstan-0 + tests-groen. ⇒ PR-CI kan tijdelijk niet de merge-gate zijn; gate op lokale verificatie tot de token-secret ververst is (alleen de gebruiker kan org/repo-secrets zetten). framework PR #3 faalt daarnaast terecht op de 10 WIP-phpstan-fouten (release = de v0.5.6/v0.5.7-tags, niet PR #3). Bugfix-PR's open+mergeable: channel-google-drive#5 (Content-Type), app-standards#2 (passwords.enabled). Mergen zelf = beslissing van de gebruiker (outward-facing); niet auto-gemerged.

**Openstaand:**
- **CI-infra: `MORTELOS_CI_TOKEN`-secret verversen** zodat de phpstan-workflows weer `composer install` kunnen (nu rood op auth, niet op code).
- Boundary-PR's (channel #4) mergen ná hun slice-1-PR's (ze stacken op test-hardening). **Channels die slice-2 mergen moeten naar framework v0.5.7.**
- Overige slice 2: ~~entity-graph Actions/HTTP~~, ~~document-studio sign-off~~, ~~mail (sqlite)~~, ~~chat DOCX/CSV~~, ~~widget-compliance render~~ (alle gedaan, 2026-06-26/27). Resterend als latere slice: mail pgsql-tsvector, chat pdf/xlsx (externe binaries), Livewire-render (policy-studio governance / widget-document-feedback / ui / daily-planner — slice-1 dekt de kern), agent-tool-execute (entity-graph).
- Bevindingen nog te fixen: ~~google-drive Content-Type~~ (gefixt), ~~policy-studio "zet X uit"-deny~~ (gefixt: order-aware regel + test), ~~app-standards passwords-gate~~ (gefixt).
- os Fase 0b PR #24 mergen (na `composer update mortelos/dev-tools` → v0.1.3).

## Release-stappenplan

Twee packages leveren *consumeerbare* nieuwe code (niet alleen tests) waar anderen van afhangen; die bepalen de volgorde. De overige 20 PR's zijn test-only en hebben geen onderlinge afhankelijkheden.

**Kritiek pad (volgorde telt):**

1. **framework** PR #3 → release **v0.5.5** (huidig: v0.5.4). Levert `Mortel\Testing\TenantTestCase` (slice-2-harnas).
   - ⚠️ PR #3 bevat ook user-commit `176c7e7` (InboxDetailTool-WIP, 2 falende tests). Vóór tag: óf die WIP fixen/afmaken, óf de harnas+test-commits cherry-picken naar een schone branch en die taggen — zodat de release groen is.
   - `git checkout main && git merge test-hardening` (of cherry-pick) → `git tag v0.5.5 && git push origin v0.5.5`.

2. **dev-tools** PR #2 → release **v0.1.3** (huidig: v0.1.2). Levert `mortelos:package-tests:check`.
   - Merge → `git tag v0.1.3 && git push origin v0.1.3`.

3. **os Fase 0b** PR [uteq/os#24](https://github.com/uteq/os/pull/24).
   - In os: `composer update mortelos/dev-tools` (pakt v0.1.3) → merge #24. Gate draait dan warn-only in `quality-gates`.

4. **Slice-2 boundary rollout** (na framework v0.5.5).
   - channel-telegram: `composer update mortelos/framework` → merge branch `slice-2-boundary` (14 tests, al groen-bewezen via path-repo). Open er evt. een PR voor zodra de harnas-release live is.
   - Daarna hetzelfde patroon (boundary-tests met `TenantTestCase`) voor gmail, google-drive, moneybird, fireflies, plaud.

**Bulk (parallel, geen volgorde):**

5. De 20 test-only PR's mergen + taggen met een patch-bump wanneer het uitkomt (bv. channel-telegram v0.2.0 → v0.2.1). Geen onderlinge afhankelijkheden.

**Per-tag template:**
```bash
git checkout main && git pull
git tag vX.Y.Z && git push origin vX.Y.Z
```

## Bijwerken

Zet de Status-kolom bij aanvang op 🔨 en bij groene `composer test` + verify op ✅. Houd de
Linear-issues in sync (1 issue per package, gegroepeerd per golf).
