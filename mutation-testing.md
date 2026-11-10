# Package mutation-testing — mortelos

> Meten van *test-effectiviteit* per mortelos-package met `pest --mutate`. Opgesteld 2026-06-30.
> Eigenaar: Uteq. Vervolg op [`test-hardening.md`](./test-hardening.md) — lees die als context.
> **Status:** twee pilots (`channel-telegram`, `entity-graph`). Toolkeuze gecorrigeerd naar `pest --mutate`
> (Infection bleek vals 100% te geven op Pest-packages). `channel-telegram` opgetild naar **82% op de logica**
> met een gepinde gate (`--min=80`, console-command uitgesloten als glue). entity-graph baseline **36%**,
> nog op te tillen. Zie Voortgang.

## Waarom

Test-hardening levert *groene* suites per package. Maar groen ≠ betekenisvol: een test kan
meelopen zonder ook maar iets af te dwingen. Het test-hardening-traject ving dat tot nu toe op met
een **adversariële verify**-stap (sceptische agents die per package checken of de tests echt gedrag
toetsen — geen `assertTrue(true)`, mocks niet over-gespecificeerd).

> **Mutation testing is de machinale, herhaalbare versie van die adversariële verify.** Infection
> muteert de productiecode (verandert `>` in `>=`, gooit een `return` weg, draait een conditie om) en
> draait je suite opnieuw. Valt geen test om, dan is die mutant *ontsnapt*: je code kan stuk zónder
> dat een test klaagt. Een ontsnapte mutant wijst exact de vals-groene plek aan.

Dat sluit aan op het root-cause-first-principe: een ontsnapte mutant is de oorzaak (een test die
niets toetst), niet het symptoom. We fixen de test, niet het cijfer.

**Niet** een extra testlaag, **wel** een diagnose-instrument. Het loont daarom alléén op packages
die al groen zijn én fatsoenlijke regelcoverage hebben — anders krijg je een berg ontsnapte mutants
die je al wist (ruis, geen signaal). Daarom liften we mee op het bestaande golf-model en de
status-tabel uit `test-hardening.md`.

## De norm

> **Toolkeuze gecorrigeerd na de pilots (2026-06-30).** De eerste opzet koos Infection voor álles.
> Dat werkt **niet** voor de Pest-packages: Infection 0.34 ondersteunt Pest niet native (alleen
> phpunit/phpspec/codeception/testo) en draait Pest via een zelf-gegenereerde phpunit-config die
> Pest's bootstrap (`uses()` → service-providers → routes) niet behoudt. Gevolg: een deel van de
> suite draait niet, faalt stilletjes, en élke mutant lijkt "gedood" → **vals 100%**. Zie Voortgang.

- **Pest-packages (verreweg de meeste):** **`pest --mutate`** (`pestphp/pest-plugin-mutate`, komt al
  mee met Pest 4). Draait door de echte Pest-bootstrap, dus de cijfers kloppen. Scope met `--everything`
  of, sneller, met `covers()`/`mutates()` in de tests.
- **`framework` (kale PHPUnit 12, geen Pest):** **[Infection](https://infection.github.io)**. Dáár is
  Infection wél correct en betrouwbaar.
- **Coverage-driver:** **pcov** in CI (veel sneller; `setup-php` met `coverage: pcov`). Lokaal op Herd:
  xdebug via `-d zend_extension`, en **non-parallel** — `--parallel` spawnt workers zonder de driver
  (zie de Herd-valkuil onderaan).
- **Gate-cijfer:** **MSI** (Mutation Score Indicator) — bij `pest --mutate` simpelweg "Score". We sturen
  op de score over gedekte code (`--covered-only`), niet op kale code-coverage; dat laatste is het
  domein van test-hardening, niet van deze gate.
- **Scripts** (per package): `mutation` → `pest --mutate --everything --covered-only` (Pest), of
  `infection --threads=max` (framework).

## Doelnorm — getrapt, niet plat 80% (onderbouwd)

Online-onderzoek (1 jul) bevestigt: er is **geen universele drempel**; de norm is getrapt en
context-afhankelijk. Concreet:

- **~80% op gedrags-/domeinlogica** (graaf-opbouw, traversal, routing, drivers, jobs). Dit is de
  gangbare "goed"-lat — Stryker's default kleurt 80% groen; Infection-gidsen noemen 85-90% op de kern.
- **~60% als acceptabele vloer op heuristiek-/glue-code** (scoring met magische drempels, console,
  resources, providers). Stryker's default behandelt 60-80% als gele waarschuwingszone, niet als fout.
- **100% nooit najagen.** Equivalente mutanten zijn 4-39% van alle mutanten en *onbeslisbaar* (niet
  automatisch te elimineren), dus een plafond onder 100% is normaal. Erkend anti-patroon: "teams die
  100% najagen schrijven tests die bewijzen dat een mutant onbereikbaar is, niet tests die bugs vangen."

Bronnen: [Stryker thresholds](https://stryker-mutator.io/docs/stryker-net/configuration/) ·
[Infection in grote projecten](https://alejandrocelaya.blog/2018/02/17/mutation-testing-with-infection-in-big-php-projects/) ·
[Infection als architectuur-enforcement 2026](https://dev.to/gabrielanhaia/mutation-testing-as-architecture-enforcement-infection-in-2026-318c) ·
[equivalente mutanten: rates + onbeslisbaarheid](https://arxiv.org/html/2408.01760v1).

## De gate

Bewust **gefaseerd** — een te hoge drempel ineens blokkeert elke PR:

1. **Meten, niet blokkeren.** Eerst per package de baseline-Covered-MSI vaststellen en in de
   status-tabel zetten. Geen CI-failure.
2. **Drempel = baseline.** `--min-covered-msi` vastzetten op de gemeten baseline (afgerond naar
   beneden). Vanaf dan kan het cijfer alleen nog omhoog — een PR die de tests verzwakt, faalt.
3. **Per golf optrekken.** Bij elke hardening-slice de drempel een stap omhoog, tot een per-package
   streefwaarde (richtgetal, geen dogma: ~80% Covered-MSI voor kernlogica, lager mag voor
   presentatie/glue).
4. **CI draait diff-only op PR's.** `--git-diff-filter=AM --git-diff-base=origin/main` zodat alleen
   gewijzigde regels gemuteerd worden → CI-tijd blijft hanteerbaar. De volle run draait periodiek
   (push naar main of nightly), niet op elke PR.

Drempels staan **per package** (in `infection.json5`), nooit globaal — packages verschillen te veel.

## Pilot — channel-telegram

Zelfde pilot als test-hardening (al 🟢, klein: 7 src-files, 23 tests groen). Doel: de hele keten één
keer opzetten en als sjabloon vastleggen.

Src-oppervlak:

```
src/Console/TelegramHealthCommand.php
src/Console/TelegramSetupCommand.php
src/Http/Controllers/TelegramWebhookController.php
src/Jobs/ProcessInboundTelegramJob.php
src/TelegramApiClient.php
src/TelegramChannelServiceProvider.php   ← kandidaat voor exclude (DI-bedrading, weinig logica)
src/TelegramDriver.php
```

Stappen:

1. `composer require --dev infection/infection` in `channel-telegram`.
2. pcov regelen (lokaal via de PHP-build; in CI via `shivammathur/setup-php` met `coverage: pcov`).
3. `infection.json5` aanmaken (source `src`, `@default`-mutators, Pest, logs naar `build/infection/`,
   ServiceProvider in `source.excludes` mits muteren daar geen waarde toevoegt).
   `phpunit.xml` heeft nu geen `<source>`-blok — niet strikt nodig (Infection gebruikt z'n eigen
   `source.directories`), maar handig om toe te voegen voor consistente coverage-scoping.
4. **Baseline meten:** `vendor/bin/infection --threads=max --show-mutations`. Covered-MSI noteren.
5. **Ontsnapte mutants beoordelen** (de kern van de oefening). Per ontsnapte mutant: óf een test
   toevoegen die hem vangt (root-cause), óf hem expliciet negeren *met reden* (`@infection-ignore`
   of config) als het een equivalente/irrelevante mutant is. Niet het cijfer masseren.
6. **Drempel vastzetten:** gemeten Covered-MSI als `--min-covered-msi` in de config.
7. **CI-job toevoegen:** aparte `mutation.yml` (of stap in `ci.yml`) met pcov + diff-filter op PR's.
8. **Sjabloon vastleggen** in dit doc (de `infection.json5` + CI-stap als kopieerbaar blok) — net
   zoals het skelet-sjabloon bij test-hardening.

Daarna pas uitrol per golf.

## Uitrolvolgorde

Inventaris (30 jun): **19 Pest-packages** → `pest --mutate`; **6 PHPUnit-packages met tests** → Infection
(`app-standards`, `daily-planner`, `dev-tools`, `framework`, `issue-factory`, `package-registry`).
`channel-google-calendar` heeft 0 tests → eerst hardenen (valt buiten deze uitrol).

**Twee golven, want het zijn twee verschillende soorten werk** (de pilots bewezen dat het verschil groot is):

### Golf 1 — tooling + baseline meten (goedkoop, breed, ~30 min/package)
Mechanisch en uniform over álle groene Pest-packages: `composer mutation` + `mutation.yml` (pcov,
`--dirty` op PR, informatief — **geen `--min`**) toevoegen en de baseline-score meten + in de tabel
hieronder zetten. Levert in één keer de complete kaart van waar de tests vals-groen zijn, vóór we ergens
testwerk in steken. Geen scores optillen in deze golf.

### Golf 2 — optillen naar 80% (duur, gericht, per package)
Geprioriteerd op (a) logica-waarde en (b) hoe ver onder 80% de baseline zit. **Kosten-ijkpunt uit de
pilot:** een klein logica-package ≈ 16 tests voor ~+10 punten; de noemer-groei maakt elk punt duurder.

1. **Kernlogica (hoogste opbrengst, laagste baselines):** entity-graph (36%, gemeten), policy-studio,
   chat, mail, overviews, document-studio. Reken op tientallen tests per package, meerdere sessies.
   entity-graph eerst (al gemeten, kern-algoritmes nauwelijks gepind).
2. **Channels (telegram-vormig, sjabloon herbruikbaar, goedkoper):** gmail, google-drive, moneybird,
   fireflies, plaud. Dezelfde driver/job/controller-vorm als telegram → test-patronen kopiëren.
3. **Presentatie/standaarden (veel glue, lage plafonds):** agent-standards, ui, widget-compliance,
   widget-document-feedback, site, starter, feedback. Meet-only of zachte gate; sluit console/render-glue
   uit zoals bij telegram's HealthCommand.

### Aparte baan — `framework` (PHPUnit → Infection)
Grootste suite (49 test-files), de enige niet-Pest. Eigen Infection-opzet (zie norm). Pas oppakken nadat
het Pest-sjabloon over een paar packages bewezen is. De overige PHPUnit-packages (dev-tools, daily-planner,
issue-factory, app-standards, package-registry) volgen dezelfde Infection-baan.

### Baseline-tabel (golf 1 gemeten 30 jun)

Scores zijn **hele-pakket** (`--everything`, dus incl. console/glue). De logica-score ligt hoger — bij
telegram was hele-pakket 72% maar de logica 82%. Golf 2 scopet op de logica, dus dit is de ondergrens.

Kolom "logica-gate" = de geverifieerde `mutation:gate`-score (behavioral klassen, `--min=80`) na wave 2.
De baseline-kolom is de golf-1 hele-pakket-meting.

| Package | soort | baseline | logica-gate (wave 2) | status |
|---|---|---|---|---|
| channel-telegram | channel | 72% | **82%** | ✅ gepind, geverifieerd |
| channel-gmail | channel | 62% | **92%** | ✅ gepind, geverifieerd |
| chat | kernlogica | 77% | **92%** (heel pakket) | ✅ gepind, geverifieerd |
| channel-moneybird | channel | 69% | **89%** | ✅ gepind, geverifieerd |
| channel-fireflies | channel | 67% | **88%** | ✅ gepind, geverifieerd |
| channel-google-drive | channel | 62% | **82%** | ✅ gepind, geverifieerd |
| channel-plaud | channel | 55% | **81%** | ✅ gepind, geverifieerd |
| overviews | kernlogica | **95%** | — | alleen gate pinnen |
| agent-standards | glue | 100% (7 mut) | — | alleen gate pinnen |
| ui | glue | 100% (14 mut) | — | alleen gate pinnen |
| mail | kernlogica | 72% | **90%** | ✅ gepind, geverifieerd |
| policy-studio | kernlogica | 63% | **84%** | ✅ gepind, geverifieerd (+ 4 ongedekte domein-services: vervolg-slice) |
| document-studio | kernlogica | 62% | **85%** | ✅ gepind, geverifieerd |
| feedback | kernlogica | 58% | **83%** | ✅ gepind, geverifieerd (GitHub/Linear-clients ~76% door string-cast-plafond) |
| widget-compliance | glue | 68% | ⬜ | laag plafond (glue) |
| widget-document-feedback | glue | 52% | ⬜ | laag plafond (glue) |
| entity-graph | kernlogica | **36%** | ⬜ (matcher 35→48%) | groot; Actions killbaar (behavioral), matcher heuristiek-plafond |

**Golf-2-prioriteit op data:** entity-graph (36%) → feedback/plaud (55-58%) →
document-studio/gmail/drive/policy-studio (~62%). Channels zijn telegram-vormig (goedkoper).
overviews/agent-standards/ui zijn al klaar. De widgets hebben lage plafonds (glue).
(feedback-regressie is 1 jul gefixt — 5 tests gemigreerd naar `DefaultFeedbackDeliveryHandler`.)

## Valkuilen

- **Heuristiek-code heeft een equivalent-mutant-plafond.** Code met magische drempels (bv.
  `EntityGraphSearchMatcher::minimumScore` = 80/72/70/66/58) produceert *discrete* scores. Een mutant die
  een drempel met 1 ophoogt (`72 → 73`) is dan **equivalent**: geen zoekterm levert een score in het
  gevoelige bereik, dus geen test kan hem doden. Gemeten: de Matcher tilt met 6 gerichte tests van 35% →
  48%, en de rest is grotendeels equivalent. **Gevolg: 80% is niet haalbaar op zulke code** zonder
  ignore-annotaties (cosmetisch) of de code te verwringen (over-engineering om een metric). Streef 80% op
  *gedrags*-logica (graaf-opbouw, traversal, routing); accepteer een gedocumenteerd lager plafond op
  heuristiek-scoring. Dit is dezelfde soort grens als de equivalente caption-mutanten bij telegram, maar
  structureler.
- **Vals 100% met de verkeerde tool.** Infection op Pest-packages → de hele meting is ruis (zie
  Voortgang). Gebruik `pest --mutate` voor Pest, Infection alleen voor `framework`.
- **Traagheid.** Elke mutant draait de dekkende tests opnieuw. Mitigatie: `--covered-only`, `covers()`
  scopen, diff-only op PR's, volle run alleen op main/nightly.
- **Equivalente mutants.** Sommige mutaties veranderen het gedrag niet. Die vréét je niet weg met
  tests — expliciet negeren met reden, zodat de score eerlijk blijft.
- **Score als KPI najagen.** Het cijfer is een diagnose, geen doel. Een ontsnapte mutant die een echte
  testgat blootlegt is waardevoller dan drie procentpunten erbij. Niet het cijfer masseren om de
  gate groen te krijgen — dat is precies het symptoom-fixen dat we willen vermijden.
- **Twee testframeworks.** Pest-packages → `pest --mutate`; `framework` (kale PHPUnit) → Infection.
- **CI-secret.** Mutation-CI heeft net als de phpstan-job de `MORTELOS_CI_TOKEN` nodig voor
  composer-install van private mortelos-deps.

## Open keuzes (vóór pilot-uitvoering)

1. **CI-trigger:** diff-only op elke PR + volle run nightly, óf alleen handmatig/periodiek? (Voorstel:
   diff-only op PR, nightly vol — beste signaal/kosten.)
2. **Gate hard of zacht in fase 1:** mutation-job als *required* check, of eerst informatief
   (mag rood zijn) tot de baselines staan? (Voorstel: zacht tot alle golf-A-baselines vaststaan.)
3. **Aanhaken op de bestaande package-gate:** wil je dat `mortelos:package-tests:check` (dev-tools)
   later óók een `infection.json5` gaat eisen, zoals het nu een `test`-script eist? (Pas ná uitrol.)
4. **Tracking:** eigen Linear-project, of issues onder de bestaande *Package test-hardening*-initiative?

## Lokaal draaien op Herd (belangrijke valkuil)

De dev-machine draait **Laravel Herd**: een kant-en-klare PHP-build **zonder** `pecl` en zonder
geactiveerde coverage-driver. `PHP_INI_SCAN_DIR` wordt door de Herd-`php`-shim genegeerd, dus je kunt
xdebug niet via een env-var bijladen. Wél levert Herd xdebug-`.so`'s mee in
`/Applications/Herd.app/Contents/Resources/xdebug/` (per PHP-versie/arch, bv. `xdebug-84-arm64.so`).

Truc die Herd's config nergens aanraakt: laad xdebug per losse aanroep met `-d zend_extension` (dat
werkt; `PHP_INI_SCAN_DIR` niet). `pest --mutate` heeft de driver in-proces nodig, dus draai
**non-parallel** — `--parallel` spawnt workers die de `-d`-vlag níét erven en dan stilletjes zonder
coverage draaien (lege uitkomst):

```bash
XSO="/Applications/Herd.app/Contents/Resources/xdebug/xdebug-84-arm64.so"
XDEBUG_MODE=coverage php -d zend_extension="$XSO" vendor/bin/pest --mutate --everything --covered-only
```

Voorwaarde: `phpunit.xml` heeft een `<source>`-blok (PHPUnit 12 genereert anders geen coverage). In CI
is dit allemaal niet nodig — daar laadt `setup-php` met `coverage: pcov` de driver globaal, en mag
`--parallel` wél (workers erven de globaal geladen extensie).

**Cache-valkuil (lokaal):** een stale `.phpunit.cache/code-coverage` kan een `--everything`-run
vervuilen met mutanten uit een ánder package (cross-package coverage-lek). Symptoom: "N Mutations for M
Files" noemt meer files dan het package heeft. Doe `rm -rf .phpunit.cache` vóór een lokale `--mutate`-run.
De **scoped gate** (`--class=...`) is hiertegen bestand (muteert alleen de genoemde klassen). In CI (schone
checkout) speelt het niet.

## Voortgang (log)

**2026-06-30 — twee pilots + toolkeuze gecorrigeerd.**

*Eerste opzet met Infection — bleek vals.* Infection 0.34 toegevoegd aan `channel-telegram` en
`entity-graph` (de stack zit op symfony 8.1 / `sebastian/diff` 7 / `minimum-stability: dev`; alleen
Infection ≥0.34 resolvet daartegen). Beide rapporteerden **100% Covered MSI**. Dat bleek **onjuist**:

- Scepsis-check op entity-graph: een subtiele mutatie (`minimumScore` length-3 `72 → 71`) liet de
  suite groen — die mutant **ontsnapt**, terwijl Infection hem "gedood" noemde.
- Oorzaak: Infection's eigen initiële run draaide maar 31 van 66 tests en faalde er één
  (`DefaultEntityGraphNodePresenterTest`, route niet geregistreerd) — Pest's bootstrap ging verloren
  in Infection's gegenereerde phpunit-config. Met `--skip-initial-tests` (mijn Herd-workaround) bleef
  dat verborgen en werd élke mutant vals als gedood geteld. **Les: nooit `--skip-initial-tests`
  vertrouwen, en een grove deliberate-bug-check (de 4096-wijziging) bewijst niets over subtiele mutanten.**

*Correcte meting met `pest --mutate`* (draait door de echte bootstrap):

| Package | tests | Pest-mutate score | grootste gaten |
|---|---|---|---|
| channel-telegram | 37 ✅ | **72%** (110 untested / 285 tested) | `TelegramDriver` chunk-grenzen (`<=`, off-by-one `mb_substr`, `continue`/`break`, `&&`/`||`) |
| entity-graph | 66 ✅ | **36%** (999 untested / 556 tested) | kern-algoritmes: `BuildEntityGraph` (187), `FindEntityGraphPath` (131), `Metrics` (94), `Matcher` (90) |

Dit is het echte signaal: groene suites die de happy-path raken maar de vertakkingen/grenzen van de
logica niet vastpinnen. entity-graph's graaf-opbouw en padvinding zijn nauwelijks gehard.

**2026-06-30 — `channel-telegram` op 80% (logica) + gate gepind.**
- 16 gerichte tests toegevoegd op de échte logica: chunk-grenzen + hard-split-continue + message-id-
  doorgifte (driver), `edited_message`-pad + non-text-summary + username-fallback + metadata-volledigheid
  (job), lege-secret-guard + dispatch-argumenten (controller). Suite: 37 → **53 tests groen**.
- **Les: de score beweegt traag** omdat een nieuwe test óók nieuwe code dekt → nieuwe mutanten in de
  noemer. Alleen driver-tests: 72,15% → 72,73% (bijna vlak). Pas tests op al-gedekte code (job/controller)
  bewogen het echt: → **74,7%** hele pakket.
- **De console-command trok het platte cijfer omlaag.** `TelegramHealthCommand` = 37,5% (44 ontsnappers,
  allemaal string-opmaak). Pakket mín dat commando = **82%**. Conform de norm (meet op logica, soepeler op
  glue) is het commando bewust **buiten de gate** gehouden — niet verborgen: `composer mutation` toont nog
  steeds het volledige beeld incl. die 37,5%.
- **Gate gepind:** `composer mutation:gate` = `pest --mutate --covered-only --min=80 --class=<driver,
  apiclient,job,controller>`. Draait in CI op `main` (exit 0 bij 81,85%). PR's: `--dirty` informatief.
- **Mijn kosten-inschatting was te optimistisch:** "handvol grens-tests" werd 16 tests; de noemer-groei
  maakt elk procentpunt duurder dan verwacht. Reken voor golf-B-packages (entity-graph 36%) op fors meer.

**2026-07-01 — entity-graph wave-2 verkenning (bevestigt de getrapte norm).**
- Matcher (`Support`, heuristiek): 6 gerichte tests → **35% → 48%**. Rest is grotendeels equivalent
  (magische drempels 80/72/70/66/58; discrete scores raken de grens nooit). Plafond ~50%, niet najagen.
- Actions (`BuildEntityGraph`+`FindEntityGraphPath`, gedrag): covered-score **~38%** + 380 mutanten op
  **ongedekte** code. Mutant-types zijn **behavioral** (113× RemoveArrayItem, integer ±1, ternary/boolean-
  flips) → **killbaar**, geen equivalent-plafond. **80% hier haalbaar**, maar fors werk (veel graaf-output
  vastpinnen + ongedekte takken dekken). Bevestigt: 80% op gedrag, ~50-60% op heuristiek.

## Te doen na de pilots

- **Ombouw Pest-packages naar `pest --mutate`.** De Infection-artefacten in telegram + entity-graph
  (dep, `infection.json5`, `mutation.yml`, composer-script) vervangen; het `<source>`-blok in
  `phpunit.xml` blijft nuttig.
- **Drempel pinnen?** Pinnen op de gemeten baseline (`--min`) lockt kwaliteit maar blokkeert bij elke
  daling. Fase 1 = meten, dus nog niet gezet. Keuze.
- De vier open keuzes hierboven (CI-trigger, hard/zacht, package-gate, tracking) staan nog.

## Bijwerken

Bij uitrol wordt dit (net als test-hardening.md) de bron-van-waarheid met status-tabel per package.
