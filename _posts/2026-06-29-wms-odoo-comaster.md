---
title: "Co-Master Architecture: Splitting Operational and Financial Data Between WMS and Odoo"
date: 2026-06-29 10:50:00 +0700
categories: [System, Architecture]
tags: [wms, odoo, erp, architecture, inventory, data-integrity]
---

> **TL;DR** — Running two systems with overlapping inventory data always drifts. The co-master pattern assigns a clear owner to each data type: WMS owns operational movements (GR, GI, IQC, transfers), Odoo owns financial records (PO, invoice, cost, master data). Neither system should modify the other's domain.

---

## The Problem With Dual-Write

The instinct when building a WMS alongside an ERP is to keep both in sync by writing to both on every transaction. GR posted in WMS? Write it to Odoo too. Quantity adjusted? Update both.

This works until it doesn't. Every dual-write has a gap:

```
WMS write → SUCCESS
Odoo write → TIMEOUT (network flap at receiving dock)

Result: WMS qty = 100, Odoo qty = 0
```

In a warehouse running 30–50 GR transactions a day, a 1% network error rate means a divergence every few days. Each divergence is invisible until someone runs a reconciliation report — usually after a financial audit or a customer complaint.

The root cause isn't the network. It's the architecture: you've made correctness dependent on two writes succeeding atomically, without a distributed transaction coordinator. That's not achievable across two separate databases owned by different vendors.

---

## The Co-Master Split

The fix is to stop treating inventory as shared. Each system owns certain data types exclusively. The other system reads but never writes to those types.

| Data Type | Owner | The Other System Does |
|-----------|-------|-----------------------|
| Inventory movement (GR/GI/transfer) | **WMS** | Odoo mirrors via scheduled sync |
| Stock quantity (on-hand) | **WMS** | Odoo reads via pull; never authoritative |
| Purchase Order header + lines | **Odoo** | WMS reads for GR matching |
| Invoice / accounting entries | **Odoo** | WMS never touches |
| Item master (description, UOM, category) | **Odoo** | WMS caches, never edits |
| Cost / standard price | **Odoo** | WMS uses for valuation display only |
| Analytic codes (project, department) | **Odoo** | WMS tags transactions at post time |
| Supplier master | **Odoo** | WMS reads for GR supplier field |

Once ownership is explicit, the failure modes simplify: WMS operations can proceed even when Odoo is unreachable, because WMS doesn't need Odoo to confirm its own movements. The sync catches up when Odoo comes back.

---

## Pull Direction: Always Pull Before Push

The most common mistake in setting up the sync layer is writing to WMS first and pulling Odoo second — or pulling Odoo lazily (on demand, when a user searches for a PO).

Pull must happen upstream of every WMS write that references Odoo data:

```
Before GR:  pull PO from Odoo → validate PO exists + open
            → post GR in WMS
            → queue Odoo mirror (async)
```

Odoo is rate-limited (it's a cloud SaaS in most configurations). Burst-writing to Odoo after every WMS event triggers 429s and temporary bans. The scheduler should run on a fixed cadence — every 15 minutes is enough for most manufacturing operations — rather than per-event.

If a GR is posted against a PO that hasn't been pulled yet, you get an orphan record: the WMS knows the GR happened but can't link it to a financial document. These accumulate silently and surface only at month-end reconciliation.

---

## The Verify Page

Drift between WMS and Odoo is not always an error. Odoo-side operations — MO component consumption, customer returns, inventory adjustments made directly in Odoo by accounting — move stock without WMS knowing. The sync runs on a schedule, so there's always a window.

The solution isn't to prevent drift; it's to make it visible.

A `/verify` page compares WMS `on_hand` quantity vs `stock.quant` from Odoo for each SKU. It shows:
- Items in sync (green)
- Items with expected lag (within sync window — grey)
- Items with unexplained divergence (red — needs investigation)

The key distinction is **drift = expected** vs **drift = a bug**. A SKU that Odoo consumed via an MO two hours ago and WMS hasn't synced yet is expected drift. A SKU where WMS shows 100 and Odoo shows 0 with no recent Odoo operations is a bug.

The verify page is a reconciliation surface, not an error dashboard. Showing it to operators without context causes false alarms. Show it to the system team on a schedule.

---

## What Breaks Co-Master

**Direct database edits.** A developer who fixes a quantity discrepancy by running `UPDATE t_inventory_uids SET qty = 100 WHERE sku = 'ABC'` has bypassed every sync hook. The WMS now shows 100; Odoo still shows whatever it showed before; the movement has no audit trail.

**Odoo batch operations without sync notification.** When a production planner runs an MO that consumes 500 components in Odoo, the sync scheduler needs to pick this up within its window. If the scheduler is broken or paused, the divergence grows with every MO.

**WMS reading Odoo accounting fields.** Odoo's accounting module exposes invoices, journal entries, and payment state through its API — but the field names and access rules change between Odoo versions, and the API rate limits apply. Attempting to surface payment status in WMS by querying Odoo accounting fields creates a fragile dependency. Accounting state belongs in the AP tracker, not the WMS.

---

## Sunset Vision

The co-master architecture is built with Odoo removability in mind. WMS data structures use Odoo references (`odoo_po_id`, `odoo_partner_id`) as foreign metadata, not as primary keys. If Odoo is replaced with another ERP or a custom system, the WMS continues operating and the sync layer gets a new adapter — the WMS schema doesn't change.

This matters because ERP migrations are expensive and disruptive. Any WMS that hardcodes Odoo model IDs as primary keys is effectively locked to Odoo. The co-master split, properly implemented, keeps the options open.
