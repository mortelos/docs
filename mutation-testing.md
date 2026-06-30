# Package mutation-testing — mortelos

> **Plan** (nog niet uitgevoerd) voor het meten van *test-effectiviteit* per mortelos-package
> met Infection. Opgesteld 2026-06-30. Eigenaar: Uteq.
> Vervolg op [`test-hardening.md`](./test-hardening.md) — leest die als context.

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

- **Tool:** [Infection](https://infection.github.io) (`infection/infection`, dev-dependency per
  package). Werkt zowel met **Pest 4** (de meeste packages) als met **kale PHPUnit 12** (`framework`).
- **Coverage-driver:** **pcov** (veel sneller dan xdebug voor Infection). Lokaal en in CI vereist —
  Infection heeft regel-coverage nodig om te weten welke mutanten gedekt zijn. De huidige CI draait
  `coverage: none`; de mutation-job zet dat aan.
- **Config per package:** een `infection.json5` met:
  - `source.directories: ["src"]`
  - `mutators: { "@default": true }` als startpunt (later verfijnen, niet vooraf).
  - `logs.text` + `logs.html` naar `build/infection/` (gitignored).
  - `testFramework: "pest"` (of `"phpunit"` voor framework).
- **Gate-cijfer:** **MSI** (Mutation Score Indicator) en **Covered-MSI**. We sturen op
  **Covered-MSI** (kwaliteit van de tests die er zijn), niet op kale MSI (die straft ook ongedekte
  code af — dat is het domein van test-hardening, niet van deze gate).
- **Scripts** (per package, naast de bestaande): `mutation` → `vendor/bin/infection --threads=max`.

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

## Golven (uitvoervolgorde)

Hergebruikt de golven uit `test-hardening.md`, maar **alleen op packages die daar 🟢/✅ zijn** — de
🔨/⬜-packages eerst hardenen, dan pas muteren.

- **Golf A — externe-API channels:** telegram (pilot) → gmail, google-drive, moneybird, fireflies, plaud.
- **Golf B — kernlogica:** entity-graph, policy-studio, chat, overviews, document-studio, mail.
  Hoogste verwachte opbrengst (meeste echte logica → meeste betekenisvolle mutants).
- **Golf C — presentatie/standaarden:** agent-standards, ui, widget-compliance, widget-document-feedback.
  Lagere streef-MSI; veel glue-code waar mutanten weinig zeggen.
- **Golf D — bestaande suites:** framework (PHPUnit-variant van de config!), feedback, issue-factory,
  daily-planner, app-standards, dev-tools.

## Valkuilen

- **Traagheid.** Infection draait de suite per mutant. Mitigatie: `--threads=max`, `--only-covered`,
  diff-only op PR's, volle run alleen op main/nightly.
- **Equivalente mutants.** Sommige mutaties veranderen het gedrag niet (bv. een log-volgorde). Die
  vréét je niet weg met tests — expliciet negeren met reden, zodat de drempel eerlijk blijft.
- **MSI als KPI najagen.** Het cijfer is een diagnose, geen doel. Een ontsnapte mutant die een echte
  testgat blootlegt is waardevoller dan drie procentpunten erbij. Niet het cijfer masseren om de
  gate groen te krijgen — dat is precies het symptoom-fixen dat we willen vermijden.
- **Twee testframeworks.** De config verschilt één regel (`testFramework`), maar de framework-package
  (kale PHPUnit) krijgt z'n eigen sjabloon-variant.
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

## Bijwerken

Dit doc is het plan; bij uitvoering wordt het (net als test-hardening.md) de bron-van-waarheid met
status-tabel en voortgangslog. Tot die tijd: plan-status.
