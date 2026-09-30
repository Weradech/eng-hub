---
title: "Dual-Ledger Reconciliation: Explain Stock Differences Before Correcting Them"
date: 2026-08-26 10:00:00 +0700
categories: [System, Architecture]
tags: [wms, erp, data-integrity, reconciliation, architecture, inventory]
---

> **TL;DR** — Compare WMS and ERP stock only after defining the same item, location, ownership, status and cut-off time. Preserve both source snapshots and classify each difference before anyone considers an adjustment.

This is a **read-only investigation method**. The earlier version asserted that drift was inevitable and that three mechanisms explained over 95% of discrepancies, without a linked dataset or query. Those claims are withdrawn. No write to Odoo or other accounting records is described or authorized here.

## Define what is being compared

WMS may record physical locations, containers and inspection holds while ERP may record financial or stock-accounting states. The two systems can legitimately group the same material differently. Create a mapping for product ID, unit of measure, warehouse, owner, lot, stock status and timestamp. Decide whether the comparison is for physical on-hand, available-to-pick, inspected/accepted or financially posted quantity. Do not mix those definitions in one “difference” number.

Capture read-only snapshots at a controlled cut-off time and preserve query parameters, source revision, time zone and extraction results. If one system is unavailable or the snapshot windows differ, mark the comparison incomplete. A total equal across all items can conceal offsetting item-level errors.

| Difference class to investigate | Evidence to inspect |
|---|---|
| Timing | Receipt or issue entered in one system but not yet posted in the other at the cut-off. |
| Status mapping | IQC hold, quarantine, reservation or work-in-process counted in one figure only. |
| Identity mapping | Different SKU, location, lot, owner or unit of measure. |
| Duplicate or missing movement | Source event IDs and retry history; check idempotency before replaying. |
| Manual change | Authorized adjustment record and movement history. |

These are **possible classes**, not a measured ranking of causes. A cancelled or reissued PO may complicate references, but its effect depends on the configured ERP/WMS workflow and must be traced from actual records. Do not treat a plausible story as the root cause.

## Reconcile by evidence, then decide

For each affected item and location, trace opening balance plus receipts minus issues and adjustments to the closing balance in each source. Match movement IDs and document missing, duplicate or differently classified events. Retain a list of unmatched movements and the person assigned to investigate them. If an automated sync has a retry or backfill path, replay only with the approved idempotency and audit controls for that system.

Do **not** make a balancing stock adjustment merely to force equality. Such a write can hide the real movement and create a later reversal. A correction, if required, belongs to the authorized owner of the affected ledger and must include its source evidence, approval and accounting effect. This article performs no Odoo write.

## Report the result without claiming closure early

Show the checked scope, matched items, open differences, classification, owner and next check time. A machine report can say “no difference found within this comparison”; it cannot prove physical stock accuracy, complete movement history or production readiness. Where physical stock matters, reconcile to a controlled count and its variance disposition. See [reconciliation without false alarms]({% post_url 2026-06-13-reconciliation-without-false-alarms %}) and the [financial versus operational ownership model]({% post_url 2026-05-27-financial-core-vs-operational-core %}) for the boundaries.
