---
title: "AP Tracker Pattern: Reconcile PO, Receipt and Invoice Evidence"
date: 2026-06-29 11:00:00 +0700
categories: [System, AP]
tags: [ap-tracker, procurement, erp, wms, workflow]
---

> **TL;DR** — A tracker can show which purchase-order lines have receipt and invoice evidence, and which exceptions need review. It must preserve source ownership and leave payment authorization and accounting status with the actual finance process.

This article describes a **proposed integration pattern**, not a verified deployment or an achieved automation rate. The PO, goods receipt and invoice records must be inspected in the systems that own them. Access to Odoo for this pattern is read-only unless the owner explicitly authorizes a separate write workflow.

## Reconcile lines, not just a PO number

A PO number is a useful starting key, but one PO can have multiple lines, deliveries, invoices, units of measure and revisions. A supplier may reference the PO differently on a delivery note. Build a mapping using stable source IDs and line IDs, then retain the human-readable numbers for investigation. Do not infer a unique match from supplier and date alone.

| Record | Source-owned facts to preserve |
|---|---|
| Purchase order | PO and line IDs, item, ordered quantity, unit, price, currency, terms and approved revision. |
| Goods receipt | GR and line IDs, actual quantity, accepted/held/rejected status, lot or serial and posting time. |
| Invoice | Invoice and line IDs, billed quantity and amount, tax, currency, credit notes and finance status. |

The tracker can calculate a **candidate** match and show discrepancies. A partial receipt, over-delivery, quality hold, duplicate invoice or currency mismatch stays open for the responsible person. An equal PO and GR quantity alone does not mean an invoice is valid or payable.

## Keep statuses narrow

Use explicit states such as `PO_SEEN`, `RECEIPT_PARTIAL`, `RECEIPT_COMPLETE` and `EXCEPTION_REVIEW`. Derive each from a documented source query and calculation. Record the last successful sync time, source IDs and any missing data. If a source is unavailable, show the record as stale or unverified; do not silently keep a green state.

`READY_FOR_REVIEW` may mean that the tracker has assembled evidence for a human. It must not mean payment approved. The finance system and its authorized approver own invoice validation, payment approval, posting and bank reconciliation. A dashboard button is not proof that money was transferred.

## Validate before automation

Start with a small read-only sample containing normal, partial, duplicate and held receipts. Compare the tracker's line-level matches with the source records and document false matches and misses. Only automate a match rule after its exact scope and exception handling have been reviewed. No percentage of automatic matches or five-minute freshness is claimed here; measure those on the actual configured system if they matter.

For the broader data-ownership boundary, see [financial versus operational core]({% post_url 2026-05-27-financial-core-vs-operational-core %}) and [reconciliation without false alarms]({% post_url 2026-06-13-reconciliation-without-false-alarms %}).
