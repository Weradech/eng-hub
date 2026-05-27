---
title: "PCBA RFQ Costing Pattern — How to Price a PCBA Assembly Professionally"
date: 2026-05-27 09:00:00 +0700
categories: [NPI, RFQ]
tags: [pcba, costing, rfq, bom, npi, ems]
---

> **TL;DR** — 6-step PCBA costing from BOM to Quotation: Material → Assembly → OH → NRE → Margin → Final Price, with production-proven rules of thumb.

---

## Why Accurate Costing Matters

In PCBA manufacturing, a 5–10% error on material cost can wipe out an entire project margin. EMS/OEM margins typically sit at 15–25%, leaving almost no room for miscalculation.

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

**Rate reference (Thai EMS, 2026):**

| Type | Rate |
|------|------|
| SMD 0402+ | ฿0.30–0.50 / placement |
| SMD 0201 | ฿0.80–1.20 / placement |
| Through-hole | ฿1.50–3.00 / placement |
| Machine Setup | ฿17,500 / lot |

---

### Step 3 — Overhead (OH)

> **Rule: OH = 15% applied on manufacturing costs only**

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
Selling Price = (Material + Assembly + OH) × (1 + Margin%)
             + NRE (lump sum, separate line)
```

**Margin guideline:**
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

## Case Study — CDH1131A / LDU M

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
Final Price = [(Material + Assembly) × 1.15] / (1 − Margin%) + NRE/lot
```

This pattern scales from 5-piece prototypes to 10,000-piece production runs — adjust only the margin and MOQ-adjusted material price.
