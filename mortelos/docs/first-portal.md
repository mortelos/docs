---
title: "First Portal"
nav_title: "First portal"
slug: "first-portal"
version: "0"
description: "Plan the first useful vertical slice for a MortelOS portal."
section: "getting-started"
order: 20
status: "mvp"
audience: "developers"
canonical_path: "/docs/0/first-portal"
last_verified: "2026-06-07"
public: true
---

# First Portal

A MortelOS portal starts with capabilities, not pages.

Ask what each role must be able to do. Then map the data, approvals, workflows and surfaces needed to make that capability safe.

## Capability map

Write a short capability map before implementation.

| Area | Question |
| --- | --- |
| Goal | What outcome must the portal make possible? |
| Roles | Who uses it and what trust level do they have? |
| Read capabilities | What may each role view or search? |
| Write capabilities | What may each role create, upload, edit, approve or trigger? |
| Data | Which domain objects matter? |
| External systems | Which systems provide or receive data? |
| Approvals | Which actions need human review? |
| Risk | Which data or action is sensitive? |
| Surfaces | Where should users complete the work? |
| Reuse | Could this serve another MortelOS installation? |

Keep the map short enough for a stakeholder to review. Unknowns that do not block the first slice can be follow-ups. Unknowns that affect tenant isolation, approvals or sensitive writes block implementation.

## Package decision

Record the package boundary before adding a surface, workflow, connector, widget or reusable service.

| Decision | Use when |
| --- | --- |
| `package-now` | The capability can serve another MortelOS installation today. |
| `package-ready` | The host needs local wiring first, but the package boundary is explicit. |
| `workspace-only` | The behavior is specific to one customer or workspace. |

Default to `package-ready` when unsure. It preserves speed while keeping the future package boundary visible.

## First vertical slice

Choose the smallest useful end-to-end capability. A good first slice usually contains:

1. One role.
2. One domain object.
3. One read model.
4. One workflow or action.
5. One surface where the user completes the work.
6. One policy test for allowed access and one for denied access.

Do not start with the full navigation tree. Build the first capability to completion, then expand.

For a document review portal, a good first slice is:

```text
Customer uploads a document
  |
  v
DocumentUploaded event is recorded
  |
  v
DocumentReviewStatus projection shows pending
  |
  v
Account manager sees an inbox item
  |
  v
Account manager approves or rejects
  |
  v
Projection updates and audit history explains the decision
```

## Done means verified

Before claiming the first slice is done, verify:

```bash
vendor/bin/pint --dirty
php artisan starter:doctor
vendor/bin/pest
```

Also run the browser smoke manually:

```text
Open http://127.0.0.1:8000
Log in as admin@example.test / password
Reach dashboard
Open the new capability surface
Verify allowed and denied role behavior
```

For a capability with approvals, verify both approve and reject paths.
