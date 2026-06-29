---
title: "AP Tracker: Automating the PO→GR→Payment Lifecycle"
date: 2026-06-29 11:00:00 +0700
categories: [System, AP]
tags: [ap-tracker, procurement, erp, automation, wms, workflow]
---

> **TL;DR** — The accounts-payable cycle spans 3 systems and 4 manual steps. AP Tracker closes the loop: PO lands in Odoo → WMS records GR → AP Tracker auto-imports both → matches them → surfaces discrepancies before payment is due. Stakeholders get a status dashboard instead of chasing emails.

---

## The Problem This Solves

Before AP Tracker, the reconciliation workflow looked like this:

1. Purchasing creates a PO in Odoo
2. Warehouse receives goods and posts a GR in WMS
3. AP clerk downloads a PO report from Odoo and a GR report from WMS
4. Clerk reconciles manually in Excel — matching PO lines to GR lines by part number and quantity
5. Clerk emails accounting: "PO-2026-1234 received in full, OK to pay"
6. Accounting approves payment

Steps 3–5 happened once or twice a week, in batch. Mismatches — partial deliveries, wrong quantities, items received without a PO — sat invisible until the batch run. Items that were never received but appeared on invoices didn't surface until the invoice was already approved.

The spreadsheet was the integration layer. The AP Tracker replaced the spreadsheet.

---

## Architecture

AP Tracker is a Next.js frontend served from the same host as the WMS, with backend routes under `/srm`. It reads from two sources:

- **Odoo**: PO headers and lines, supplier data, payment terms — via the Odoo XML-RPC API (same connection pool as the WMS Odoo bridge)
- **WMS**: GRN records from the stock card ledger — via internal WMS API calls, not direct DB

The join key across all three systems is the **PO number** — a human-readable string like `P02026-001234` that Odoo generates and operators write on physical delivery notes. WMS GR records capture this number at receiving time.

No new database schema was needed. The AP Tracker maintains a lightweight state table (PO ID → status → linked GRN ID → timestamps) but never stores the financial or inventory data itself — it reads both systems live.

---

## Auto-Import Flow

A cron job runs every 5 minutes and polls Odoo for POs in `purchase` or `done` state modified in the last 24 hours:

```
cron (every 5 min)
  → fetch Odoo POs modified since last run
  → for each PO:
      look up GRN in WMS by po_no
      if GRN found:  create/update AP record → status RECEIVED
      if not found:  create AP record → status PENDING_GR
  → write state table
```

No user action triggers this. New POs appear in the AP Tracker within 5 minutes of being confirmed in Odoo. GRs appear within 5 minutes of being posted in WMS.

---

## Auto-Transition States

Each AP record moves through states without manual input (except at the financial gate):

```
PENDING_GR      → RECEIVED          (GRN linked, any quantity)
RECEIVED        → READY_FOR_PAYMENT (GR qty ≥ PO qty for all lines)
READY_FOR_PAYMENT → DONE            (human approves payment — manual gate)
```

Partial deliveries stay in `RECEIVED` and surface a flag: `partial — 80/100 units`. The SCM reviewer sees this immediately rather than discovering it at payment time.

The payment confirmation step is intentionally manual. Automating a financial approval requires controls that are out of scope for an internal ops tool. The AP Tracker makes the decision obvious; a human makes the decision.

---

## The GRN Auto-Discover Pattern

When a new GR is posted in WMS, the `OtsStockCard` ORM watcher fires. AP Tracker receives a notification and runs a match search:

1. Filter open AP records by supplier (from PO) matching GR supplier
2. Filter by date window (GR date within ±7 days of PO expected delivery)
3. Filter by part number overlap (at least one matching SKU)

If this returns **exactly one** PO: auto-link, status → RECEIVED.  
If it returns **zero or multiple**: surface for manual review with the candidate list pre-populated.

The auto-discover handles ~85% of GRs in practice. The remaining 15% are split-deliveries, multi-PO shipments, or GRs posted without a PO reference — all of which need human judgment anyway.

---

## What Doesn't Automate

**Payment confirmation.** Financial approval is a human gate. Even when GR qty matches PO qty exactly, payment requires a person to confirm the invoice amount, check payment terms, and approve. AP Tracker routes to the right approver; it doesn't approve.

**Partial deliveries.** When 80 of 100 units arrive, AP Tracker surfaces the shortfall but doesn't decide: wait for the remaining 20, pay for 80 now, or request a credit note. That's a commercial decision between Purchasing and the supplier.

**Cross-currency.** POs in USD or EUR need FX conversion at the PO-confirmation rate, not today's rate. AP Tracker stores the original currency and amount; it doesn't compute the THB equivalent because the exchange rate is locked to the PO date and must match the accounting entry exactly.

**Odoo accounting fields.** Odoo's payment state, journal entries, and bank reconciliation data are not accessible via the standard XML-RPC API in most configurations. AP Tracker can't read "was this invoice actually paid?" from Odoo — it only knows "a human clicked DONE in AP Tracker." The accounting team maintains the authoritative payment record in Odoo.

---

## Stakeholder Map

| Role | Name | Responsibility in AP Tracker |
|------|------|------------------------------|
| SCM | มุก | Reviews PENDING_GR items, resolves partial delivery flags, escalates supplier issues |
| Accounting | นาเดีย | Approves READY_FOR_PAYMENT, confirms DONE after bank transfer |
| System | MIS / Iris | Monitors cron health, investigates auto-discover misses |

AP Tracker does not send autonomous notifications. No email, no line, no Discord. Stakeholders open the dashboard on their own cadence. The data is always current; the push channel was removed when it created noise rather than signal.
