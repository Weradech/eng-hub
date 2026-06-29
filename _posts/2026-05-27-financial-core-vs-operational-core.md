---
title: "Financial Core vs Operational Core — A Mental Model for System Integration"
date: 2026-05-27 16:00:00 +0700
categories: [System, Architecture]
tags: [erp, wms, mrp, odoo, system-integration, architecture]
---

> **TL;DR** — ERP (Odoo) = Book of Record for money. Operational systems (WMS/MRP/MES) = Systems of Engagement for work. Both must communicate, but must never be merged into one.

---

![Two-panel mental model: Financial Core (Odoo) owns money and master data, Operational Core (WMS/MRP/MES) owns daily work, and the two sync via outbox without merging]({{ "/assets/img/2026-05-27/financial-core-vs-operational-core.svg" | relative_url }})
_Each core is authoritative for its own domain and reads &#8212; never writes &#8212; the other&#8217;s; they sync, they never merge._

## The Problem with "ERP Does Everything"

Many Thai SMEs try to make ERP handle all operations:
- Record GR in ERP → slow, not real-time
- Warehouse staff use ERP directly → high error rate

Or the opposite:
- Warehouse system works standalone → ERP accounting has no visibility

The result: **two systems with mismatched data — nobody knows which to trust.**

---

## Two-Layer Mental Model

```
┌─────────────────────────────────────────┐
│           FINANCIAL CORE (ERP)          │
│  Odoo — Book of Record                  │
│                                         │
│  PO / Bill / Payment / GL / Invoice     │
│  "Numbers used for tax and management   │
│   reporting"                            │
└──────────────────┬──────────────────────┘
                   │ sync (scheduled / event-driven)
┌──────────────────▼──────────────────────┐
│         OPERATIONAL CORE                │
│  WMS / MRP / MES / CRM / AP Tracker    │
│  Systems of Engagement                  │
│                                         │
│  GR / GI / Production / IQC / Delivery  │
│  "What the team actually does every day"│
└─────────────────────────────────────────┘
```

---

## Financial Core — ERP (Odoo)

**Authoritative on:**
- Purchase Orders — supplier contracts
- Vendor Bills — invoices
- Payments — disbursements
- General Ledger — all accounting entries
- Customer Invoices

**Rules:**
- Every money-touching transaction → must pass through ERP
- ERP is the single source of truth for financial figures
- Never modify financial records outside ERP

**Analogy:** A bank — every monetary movement must be recorded here.

---

## Operational Core — Systems of Engagement

**Authoritative on:**
- Inventory movement (GR / GI / Transfer)
- Production status (MO start / complete)
- Quality inspection (IQC pass / fail)
- Customer interaction (CRM lead / quote)

**Rules:**
- Optimize for speed, usability, and real-time visibility
- User-friendly UI → fewer errors
- Push summaries to ERP (not real-time sync on every transaction)

**Analogy:** The shop floor — customers and staff work here, but end-of-day totals go to accounting.

---

## Integration Patterns

### Sync Direction

```
Operational → Financial  (push summary)
  WMS GR complete → Odoo stock.move + vendor bill matching

Financial → Operational  (pull master data)
  Odoo: Item master, Supplier, Customer, Analytic accounts
  → pulled into WMS/MRP before use
```

### Two-Status Pattern

A single transaction can carry two independent statuses:

| Status Type | Owner | Example |
|-------------|-------|---------|
| Operational | WMS | `received` → `iqc_pass` → `in_stock` |
| Financial | Odoo | `draft` → `posted` → `paid` |

These are not required to be 1:1. For example:
- WMS: `iqc_pass` (goods are in the warehouse)
- Odoo Bill: still `draft` (Accounting hasn't posted yet)

---

## Integration Anti-patterns

| Anti-pattern | Problem |
|-------------|---------|
| WMS reads Odoo stock directly on every transaction | High Odoo load, high latency |
| WMS writes directly to Odoo GL | Bypasses accounting rules, fails audit |
| Full real-time sync on every field | WMS goes down when Odoo is unavailable |
| No sync log | No way to verify which transactions reached Odoo |

---

## Outbox Pattern for Reliability

```
WMS transaction completes
    ↓
Write to local outbox table
    ↓
Background job pushes to Odoo every 15 minutes
    ↓
Mark outbox record as sent after Odoo confirms
    ↓
On failure → retry with exponential backoff
```

**Benefit:** WMS operates independently of Odoo availability — continues working even when Odoo is down.

---

## Summary

> Ask yourself before every design decision:
> - "Does this transaction touch money?" → must go through ERP
> - "Is this what the team actually does operationally?" → belongs in Operational system
> - "Does both systems need to know?" → sync via outbox pattern

Clear layer separation = scalable + maintainable + audit-ready.
