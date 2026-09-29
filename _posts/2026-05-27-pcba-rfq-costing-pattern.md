---
title: "PCBA RFQ Costing Pattern — How to Price a PCBA Assembly Professionally"
date: 2026-05-27 09:00:00 +0700
categories: [NPI, RFQ]
tags: [pcba, costing, rfq, bom, npi, ems]
---

> **TL;DR** — Build the quote from a defined BOM, assembly and test scope, current supplier prices and approved commercial rules. Separate recurring unit cost from one-time charges. The historical figures below are examples, not current supplier quotes or company pricing authority.

---

![PCBA RFQ cost build-up: material plus assembly, times 1.15 overhead, divided by one minus margin, gives the unit price; NRE is a separate lump-sum line in the quotation](/assets/img/2026-05-27/pcba-rfq-costing-pattern.svg)
_Material plus assembly, times overhead, over one-minus-margin — and keep NRE a separate line._

## Why Accurate Costing Matters

A material-cost error can change the quote materially, especially on a small lot. Check the current BOM, purchase quantity, supplier offer and currency before using a price. No universal margin or error threshold follows from this example.

---

## 6-Step Costing Pattern

### Step 1 — BOM Cost Roll-up

Sum all component costs from the BOM:

```
Total Material Cost = Σ (Unit Price × Qty per board)
```

**Watch out for:**
- Use **MOQ-adjusted** pricing, not datasheet unit price
- Remove **DNI (Do Not Install)** components before summing
- PCB bare board should always be a separate line item

---

### Step 2 — Assembly Cost

```
Assembly Cost = (Placement Count × Rate per placement) + Machine Setup
```

**Illustrative historical planning inputs, not verified 2026 market rates:**

| Type | Rate |
|------|------|
| SMD 0402+ | ฿0.30–0.50 / placement |
| SMD 0201 | ฿0.80–1.20 / placement |
| Through-hole | ฿1.50–3.00 / placement |
| Machine Setup | ฿17,500 / lot |

---

### Step 3 — Overhead (OH)

> **Example assumption:** OH = 15% of the stated manufacturing cost base. Replace this with the organization's approved cost model.

```
OH = (Material Cost + Assembly Cost) × 15%
```

⚠️ **Common mistake** — some engineers apply OH on the total including NRE. NRE is a one-time cost and must be separated before OH calculation.

---

### Step 4 — NRE (Non-Recurring Engineering)

NRE covers all one-time costs:

| Item | Note |
|------|------|
| SMT Stencil | ฿3,500–8,000 each |
| ICT / FCT Fixture | Depends on complexity |
| Wave solder jig | |
| RE Fee (Reverse Engineering) | Only when no source files available |
| Engineering hours | Rate × hours |

**For shared Laser assets:**
```
Laser NRE    = ฿0 (shared asset)
Laser per unit = ฿0.50 / unit
```

---

### Step 5 — Margin

```
Unit Selling Price = (Material + Assembly + OH) / (1 - Gross Margin%)
One-Time Charges = approved NRE, shown separately in the quotation
```

**Illustrative margin scenarios, not a recommended company policy:**
- Prototype / EVT: 25–35%
- Mass production: 15–20%
- Strategic account: negotiable

---

### Step 6 — Quotation Format

> **Golden Rule: Customer quotation = Lump sum only. Never expose OH or Margin.**

```
Unit Price : ฿XXX.XX / board
NRE        : ฿XX,XXX (one-time)
MOQ        : XXX pcs
Lead time  : X weeks
Validity   : 30 days
```

---

## Illustrative Costing Snapshot

This is a historical example with no current supplier quote or approval record attached. Recalculate every input before use on an RFQ.

| Item | Value |
|------|-------|
| BOM lines | 21 components |
| Total placements | 58 |
| Material cost | ฿94.82 / board |
| Assembly cost | ฿28.46 / board |
| OH (15%) | ฿18.49 |
| Unit price (25% margin) | ~฿190 / board |
| NRE — stencil | ฿5,500 |

---

## Python Quick Calculator

```python
import pandas as pd

df = pd.read_excel("bom.xlsx")
material  = (df["unit_price"] * df["qty"]).sum()
assembly  = df["qty"].sum() * 0.40 + 17500 / qty_per_lot
oh        = (material + assembly) * 0.15
unit_price = (material + assembly + oh) / (1 - margin)
```

---

## Summary Formula

```
Example Unit Price = [(Material + Assembly) × 1.15] / (1 − Gross Margin%)
One-Time NRE = separate quotation line
```

For each new quantity and process scope, recalculate supplier prices, setup allocation, assembly and test effort, yield/rework assumptions where applicable, and one-time charges. The [RFQ intake method]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}) records the quote basis before this calculation.
