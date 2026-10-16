---
title: "Coding Guidelines"
nav_title: "Coding guidelines"
slug: "coding-guidelines"
version: "0"
description: "Code in MortelOS packages and host apps is English. Dutch is for text a customer sees."
section: "build-method"
order: 34
status: "mvp"
audience: "developers"
package: "mortelos/dev-tools"
canonical_path: "/docs/0/coding-guidelines"
last_verified: "2026-10-16"
public: true
---

# Coding Guidelines

All code in MortelOS packages and host apps is English. Text that a customer sees stays Dutch.

`CodeLanguageGuard` from `mortelos/dev-tools` enforces the rule. It ships with dev-tools 0.1.8, and the same rule is part of the `mortelos/agent-standards` rules from 0.1.15.

## The Rule

These are English:

| Area | Examples |
| --- | --- |
| Names in code | Classes, methods, functions, variables, properties, constants, enum cases, namespaces, files |
| Data | Tables, columns, array keys, JSON keys, stored values such as enum values |
| Wiring | Config keys, environment variables, routes, events, Livewire and Blade components |
| Text around the code | Code comments, test names, commit messages, pull requests, technical docs in a repo |

Dutch is allowed in three places only:

1. **Text a customer sees.** UI copy, mail, notifications and the files in `lang/nl`.
2. **Terms that exist only in Dutch law or government.** BSN, KvK, BTW, AVG and iDEAL, plus the brand name `mortel` and `mortelos`. These sit on a fixed list in dev-tools. Adding a word takes a pull request on `mortelos/dev-tools` with a reason for each word.
3. **Strings that recognise Dutch input.** For example, synonyms a customer types into a question. Mark the string with `code-language: dutch-input <reason>`.

A session or user language setting decides how an agent talks to you. It never applies to code or code comments.

## Use The Established English Term

Pick the English word that a developer in that field would use. Do not translate word for word.

| Dutch | English | Why |
| --- | --- | --- |
| concept | `draft` | A version that is not published yet is a draft in English. |
| inhoud (of a mail) | `body` | A mail has a body. |
| normkaart | `rulebook` | A versioned set of rules per standard. |
| bronpassage | `quote` | The prompt asks for the exact quote from the transcript. |

Only a MortelOS concept gets a name of its own. When you are unsure, choose a term, and write the choice in the pull request.

## Renaming In A Released Package

Renaming a public name in a released package is a breaking change. In the 0.x series, bump the minor version and add an upgrade note. The [Changelog](/docs/0/changelog) lists the packages that did this on 2026-10-16, and each package page has its upgrade steps.

## How `CodeLanguageGuard` Works

The guard is a test helper. It reads `.php` (and `.blade.php`), `.js`, `.ts` and `.css` files under `app`, `src`, `config`, `database`, `routes`, `resources`, `tests` and `bin`. It skips every `vendor`, `node_modules`, `storage`, `public`, `lang`, `dist` and `build` directory.

It checks five rules:

| Rule | Reports a Dutch word in |
| --- | --- |
| `name` | Classes, methods, functions, variables, properties, constants, enum cases, namespaces, JS and CSS names, `wire:` and `x-` attribute values, `<livewire:…>` and `<x-…>` component names. |
| `key` | A string that looks like a key: no spaces, lower case or `UPPER_SNAKE`, with `.`, `-`, `_`, `:` or `/` allowed. This covers events, routes, tables, columns, config keys, env names and array keys. |
| `comment` | PHP, Blade, HTML, JS and CSS comments with two different Dutch words. |
| `test-name` | The first argument of `it`, `test` or `describe`. The same rule as comments. |
| `file-name` | A file name. Migrations are skipped, because apps already store those names. |

### What Counts As Customer Text

A string with a space or a capital letter is customer text and is not checked. `'Opslaan'` and `'Klant toevoegen'` pass. A string without spaces in lower case, such as `'klant_number'`, looks like a key and fails.

### Which Words Count

Names are split on camelCase, PascalCase, `_`, `-`, `.` and `/`. Each part is checked against a list of Dutch words in `resources/code-language/dutch-words.txt`. The list holds only words that are not English, so `planning`, `agenda`, `status` and `door` never report. An entry such as `stem*` also matches inside a longer word, for example `normkaart`. Words shorter than four letters only count in comments and test names.

The allowed Dutch terms live in `resources/code-language/allowed-dutch.txt`.

### The Marker

A string that recognises Dutch input takes the marker on its own line or on the line above. It covers strings only. It never covers names or comments.

```php
// code-language: dutch-input the customer asks in Dutch
$synonyms = ['klant' => 'customer', 'klanten' => 'customer'];
```

There is no allow list and no baseline. Translate the code instead.

## Add The Guard Test

Add one test to every repo, in a test folder that `phpunit.xml` or Pest runs:

```php
use Mortelos\DevTools\Testing\CodeLanguageGuard;

it('keeps code in English', fn () => CodeLanguageGuard::assertClean(dirname(__DIR__, 2)));
```

Pass the root of the repo. For a test in `tests/Feature`, that is `dirname(__DIR__, 2)`. For a test directly in `tests/`, use `dirname(__DIR__)`.

Install dev-tools as a dev dependency first. See [Dev tools](/docs/0/package-dev-tools).

```bash
composer require mortelos/dev-tools --dev
```

To give your agents the rule, publish the agent rules. See [Agent standards](/docs/0/package-agent-standards).

```bash
php artisan mortelos:agent-rules:publish
```

## Related Pages

- [Dev tools](/docs/0/package-dev-tools) for `CodeLanguageGuard` next to `ArchGuard` and `OutletsGuard`.
- [Agent standards](/docs/0/package-agent-standards) for the rule that agents receive.
- [TALL conventions](/docs/0/tall-conventions) for the UI and Livewire rules.
