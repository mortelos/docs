---
title: "MortelOS Document Studio"
nav_title: "Document studio"
slug: "package-document-studio"
version: "0"
description: "Reusable document collections, registers and sign-off packages for MortelOS."
section: "packages"
order: 59
status: "mvp"
audience: "developers"
package: "mortelos/document-studio"
canonical_path: "/docs/0/package-document-studio"
last_verified: "2026-10-16"
public: true
---

# MortelOS Document Studio

`mortelos/document-studio` provides document collections, structured registers and sign-off packages on top of `mortelos/framework` documents.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Collections | Group framework documents and registers in a collection, addressed by its slug. |
| Registers | Structured register rows per collection. The register schema's `fields` list drives the row form, and a row can link a source document. |
| Sign-off packages | Snapshot the collection's current document versions and register rows, then send each approver a pending inbox item. |
| Finalization | Once every approver has approved, lock the register rows, sign off the snapshotted document versions and set the package to `active`. |
| Document pages | Livewire pages to browse collections, add register rows, create sign-off packages, edit document Markdown into a new version, compare versions and approve packages. |
| Storage | Migrations for `document_collections`, `document_collection_items`, `document_registers`, `document_register_items` and `document_sign_off_packages`. |

The pages do not create collections or registers. Create them from host code with the `CreateDocumentCollection`, `CreateRegister` and `AddDocumentToCollection` actions. Locked register rows cannot be updated or deleted, and signed documents cannot be edited from the document page.

When `mortelos/chat` and `mortelos/widget-document-feedback` are installed, the document page embeds the `widget-document-feedback::annotator` component on a `document_feedback_annotate` widget run.

## Install

```bash
composer require mortelos/document-studio
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

## Runtime surfaces

| Surface | Key |
| --- | --- |
| Route `GET /documents` | `documents.index` |
| Route `GET /documents/collections/{collection}` | `documents.collections.show` |
| Route `GET /documents/{document}` (id or slug) | `documents.show` |
| Route `GET /documents/sign-off-packages/{package}` | `documents.sign-off-packages.show` |
| Livewire pages | `document-studio::documents.index`, `document-studio::documents.collection-show`, `document-studio::documents.document-show`, `document-studio::documents.sign-off-package-show` |
| Inbox action type | `document-studio.sign-off-package` |

A package also finalizes when its inbox item is approved outside the package page, because the package listens for the framework's `InboxItemApproved` event.

## Configuration

```bash
php artisan vendor:publish --tag=document-studio-config
```

| Key | Default |
| --- | --- |
| `document-studio.routes.enabled` | `true` |
| `document-studio.routes.middleware` | `['web', 'auth']` |
| `document-studio.sign_off.action_type` | `document-studio.sign-off-package` |

Migrations load automatically; publish them with `--tag=document-studio-migrations` if the host manages them. The page copy ships in Dutch. The package no longer publishes its views, and `OutletsGuard` fails the build on a copied view. To add your own buttons or panels to a document or collection page, register them in one of the view outlets below; to change the package's own text, open a pull request on the package (see [Outlets](/docs/0/outlets#when-no-outlet-fits)).

## Upgrading to 0.4.0

Version 0.4.0 requires `mortelos/framework:^0.7`. Upgrade the host to framework 0.7 and follow its upgrade guide before you install this release; see [Upgrading to 0.7.0](/docs/0/package-framework#upgrading-to-070).

Sign-off review inbox items now use `type: action` instead of `type: actie`. Update host filters, handlers and assertions that inspect this value. The framework 0.7 migration rewrites existing inbox rows from `actie` to `action`, so publish and run that migration when you upgrade the host.

## Outlets

An outlet is a place where you extend `mortelos/document-studio` from your app without changing the package. The table lists the outlets in its `resources/mortelos/outlets.json` as of version 0.3.1; read [Outlets](/docs/0/outlets) for how each kind works and what to do when none fits.

```bash
php artisan mortelos:outlets:list document-studio
```

| Kind | Name | What it is for | Data |
| --- | --- | --- | --- |
| view | `document-studio.document.header-actions` | On the right of a document's header, after the Markdown edit and diff buttons. Default: empty. | `document`, `version` |
| view | `document-studio.document.sidebar` | At the bottom of a document's sidebar, below Versions and Review status. Default: empty. | `document`, `version` |
| view | `document-studio.collection.header-actions` | On the right of a document collection's header. Default: empty. | `collection` |
| view | `document-studio.collection.sidebar` | At the bottom of a document collection's sidebar, below Review and sign-off. Default: empty. | `collection` |
| config | `document-studio` | Routes on or off and their middleware, the sign-off action type and the dogfood flag in `config/document-studio.php`. Publish with `php artisan vendor:publish --tag=document-studio-config`. |  |

## Boundaries

`mortelos/framework` owns documents, versions, document sign-off, inbox items and approvals. `mortelos/document-studio` owns collections, registers, sign-off packages and the pages built on them. Host apps own authentication, access policy and tenant isolation (the routes only apply the configured middleware), the `layouts::app` layout the pages render in, and the users offered as approvers. Register types and field lists are data: define them in the host when you create registers.
