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
last_verified: "2026-09-26"
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

Migrations load automatically; publish them with `--tag=document-studio-migrations` if the host manages them. The page copy ships in Dutch. To change it, publish the views with `--tag=document-studio-views`.

## Boundaries

`mortelos/framework` owns documents, versions, document sign-off, inbox items and approvals. `mortelos/document-studio` owns collections, registers, sign-off packages and the pages built on them. Host apps own authentication, access policy and tenant isolation (the routes only apply the configured middleware), the `layouts::app` layout the pages render in, and the users offered as approvers. Register types and field lists are data: define them in the host when you create registers.
