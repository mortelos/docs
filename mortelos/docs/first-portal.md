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
last_verified: "2026-05-31"
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
| Reuse | Could this serve another MortelOS installation? |

## First vertical slice

Choose the smallest useful end-to-end capability. A good first slice usually contains:

1. One role.
2. One domain object.
3. One read model.
4. One workflow or action.
5. One surface where the user completes the work.
6. One policy test for allowed access and one for denied access.

Do not start with the full navigation tree. Build the first capability to completion, then expand.

