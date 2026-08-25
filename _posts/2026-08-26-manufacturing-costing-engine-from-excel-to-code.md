---
title: "Deterministic Manufacturing Costing: Translating Packaging Geometry and BOM Rules into Code"
date: 2026-08-26 14:00:00 +0700
categories: [Engineering, Software]
tags: [costing, bom, rfq, algorithms, python, typescript, manufacturing]
---

> **TL;DR** — Contract electronics manufacturing quotations rely on intricate engineering spreadsheets with intertwined formulas: packaging clearance geometry, master carton capacity, AQL 0.65 sampling plans, and compounding material overheads. Translating these spreadsheets into a deterministic, production-grade API requires moving away from floating-point arithmetic, enforcing gross-up pricing over additive markups, and validating dual-language calculation parity between TypeScript UIs and Python backend engines.

---

![Deterministic manufacturing costing engine: translating complex spreadsheet models into precise TypeScript and Python calculation engines]({{ "/assets/img/2026-08-26/manufacturing-costing-engine-from-excel-to-code.svg" | relative_url }})
_From spreadsheets to software: converting geometry rules, compounding overheads, and financial gates into deterministic calculation pipelines._

## The Fragility of Spreadsheet-Driven Quotations

In electronics manufacturing services (EMS) and New Product Introduction (NPI), formulating a quotation for a Request for Quote (RFQ) is not a simple sum of component prices. A quote involves four interlocking cost layers:

1. **Bill of Materials (BOM) Cost:** Raw component pricing, scrap yield factors, and minimum order quantity (MOQ) penalties.
2. **Material Overhead (MOH):** Tiered logistics, import customs, and local handling fees applied specifically to raw materials.
3. **Packaging & Logistics Geometry:** Calculating the physical capacity of master cartons based on 3D PCB dimensions, component clearance, and anti-static bubble wrap.
4. **Labor & Process Engineering:** Surface Mount Technology (SMT) machine cycle time, manual insertion/soldering headcount, and In-Circuit / Functional Test (ICT/FCT) fixture time.

Historically, senior estimators maintained elaborate Excel workbooks (`Costed_BOM_Format.xlsx`) with dozens of macros and hidden cells. While flexible, spreadsheet-driven costing creates severe organizational risks:

- **Formula Inconsistency:** Different engineers tweak formulas, leading to 5–10% quotation variances for the identical PCBA.
- **Markup vs. Margin Confusion:** Accidental use of additive markup ($C \times 1.30$) instead of gross-up margin ($C / (1 - 0.30)$), severely eroding profitability on high-volume runs.
- **No Revision History:** Unclear why a quote changed between Revision A and Revision B.

---

## 1. Physical Geometry: Automated Master Carton Capacity

A common manual error in quotation is estimating packaging cost by dividing a carton price by a guessed number of units (e.g., "50 pcs per box"). 

In reality, master carton capacity is a constrained 3D bin packing problem determined by PCB dimensions, component clearance, and tray divider thickness.

```
       +-------------------------------------------------------------+
       |                     MASTER CARTON (L x W x H)               |
       |                                                             |
       |     +----------+  +----------+  +----------+                |
       |     |  PCB #1  |  |  PCB #2  |  |  PCB #3  |  ... (Grid X)  |
       |     +----------+  +----------+  +----------+                |
       |                                                             |
       |     (Grid Y)                                                |
       |     +----------+  +----------+  +----------+                |
       |     |  PCB #4  |  |  PCB #5  |  |  PCB #6  |                |
       |     +----------+  +----------+  +----------+                |
       |                                                             |
       +-------------------------------------------------------------+
```

### The Clearance and Capacity Algorithm

Let the raw PCB dimensions be Length ($L$), Width ($W$), and Thickness ($T$), with the tallest component height on top ($H_{\text{top}}$) and bottom ($H_{\text{bot}}$).

1. **Calculate Effective Dimensions with Engineering Clearances:**
   - X/Y Clearance allowance (handling margin): $c_{xy} = 2.0\%$
   - Z Clearance allowance (vertical standoff / tray cushion): $c_z = 20.0\%$

$$L_{\text{eff}} = L \times (1 + c_{xy})$$

$$W_{\text{eff}} = W \times (1 + c_{xy})$$

$$H_{\text{eff}} = (T + H_{\text{top}} + H_{\text{bot}}) \times (1 + c_z)$$

2. **Compute 3D Grid Capacity within Master Box ($B_L, B_W, B_H$):**

$$N_x = \left\lfloor \frac{B_L}{L_{\text{eff}}} \right\rfloor, \quad N_y = \left\lfloor \frac{B_W}{W_{\text{eff}}} \right\rfloor, \quad N_z = \left\lfloor \frac{B_H}{H_{\text{eff}}} \right\rfloor$$

$$\text{Total Units Per Box } (N_{\text{box}}) = N_x \times N_y \times N_z$$

3. **Packaging Cost Amortization per Finished Unit:**

$$\text{Cost}_{\text{packaging}} = \frac{\text{Cost}_{\text{carton}} + \text{Cost}_{\text{divider}} + (\text{Cost}_{\text{bag}} \times N_{\text{box}})}{N_{\text{box}}}$$

```python
import math
from decimal import Decimal

def compute_master_carton_capacity(
    pcb_length_mm: Decimal,
    pcb_width_mm: Decimal,
    pcb_total_height_mm: Decimal,
    box_length_mm: Decimal = Decimal("450.0"),
    box_width_mm: Decimal = Decimal("350.0"),
    box_height_mm: Decimal = Decimal("250.0")
) -> int:
    """
    Calculates deterministic master carton grid capacity applying
    standard +2% XY clearance and +20% Z cushioning factors.
    """
    l_eff = pcb_length_mm * Decimal("1.02")
    w_eff = pcb_width_mm * Decimal("1.02")
    h_eff = pcb_total_height_mm * Decimal("1.20")
    
    nx = math.floor(box_length_mm / l_eff)
    ny = math.floor(box_width_mm / w_eff)
    nz = math.floor(box_height_mm / h_eff)
    
    capacity = max(1, nx * ny * nz)
    return capacity
```

---

## 2. AQL 0.65 Normal Inspection Sampling Plan

Quality inspection labor is not linear. Under **ISO 2859-1 (ANSI/ASQ Z1.4) Level II Normal Sampling**, sample sizes follow strict statistical batch brackets. 

Instead of guessing inspection hours, the engine automatically embeds the standard AQL lookup table:

| Lot Size Range | Inspection Level II Code | Sample Size ($n$) | Accept ($Ac$) | Reject ($Re$) |
|:---|:---:|:---:|:---:|:---:|
| 151 – 280 | G | 32 | 0 | 1 |
| 281 – 500 | H | 50 | 0 | 1 |
| 501 – 1,200 | J | 80 | 1 | 2 |
| 1,201 – 3,200 | K | 125 | 2 | 3 |
| 3,201 – 10,000 | L | 200 | 3 | 4 |

$$\text{Total QA Labor Cost} = \frac{n \times \text{InspectionTimePerSample}}{\text{LotSize}} \times \text{HourlyQARate}$$

---

## 3. Financial Mathematics: Mark-Up vs. Gross-Up Margin

The most dangerous financial bug in custom manufacturing software is confusing **Markup** with **Gross Margin**.

```
+-----------------------------------------------------------------------------+
|                           MARKUP VS. GROSS-UP                               |
|                                                                             |
|   Direct Cost (C) = $100.00                                                 |
|   Desired Margin  = 30%                                                     |
|                                                                             |
|   [ X ] Mark-up Formula:                                                    |
|         Price = Cost * (1 + 0.30) = $130.00                                 |
|         Realized Margin = ($130 - $100) / $130 = 23.07%  (LOSS of 6.93%!)   |
|                                                                             |
|   [ V ] Gross-up Formula:                                                   |
|         Price = Cost / (1 - 0.30) = $142.86                                 |
|         Realized Margin = ($142.86 - $100) / $142.86 = 30.00% (CORRECT)     |
+-----------------------------------------------------------------------------+
```

### The Canonical Price Gross-Up Formula

When computing a sales price with multiple compounding layers (Direct Material Cost $M$, Material Overhead rate $r_{\text{moh}}$, Direct Labor Cost $L$, Manufacturing Overhead $O$, Sales Commission $r_{\text{comm}}$, and Target Margin $r_{\text{margin}}$):

1. **Compounded Direct Cost Base ($C_{\text{direct}}$):**

$$C_{\text{direct}} = \left( M \times (1 + r_{\text{moh}}) \right) + L + O$$

2. **Gross-Up Base Selling Price ($P_{\text{base}}$):**

$$P_{\text{base}} = \frac{C_{\text{direct}}}{1 - r_{\text{margin}}}$$

3. **Commission-Inclusive Final Selling Price ($P_{\text{final}}$):**

$$P_{\text{final}} = \frac{P_{\text{base}}}{1 - r_{\text{comm}}}$$

4. **Contribution Margin Hard Gate Verification:**

$$\text{Contribution Margin \%} = \frac{P_{\text{final}} - C_{\text{direct}}}{P_{\text{final}}}$$

{: .prompt-danger }
> **Business Rule Gate:** The quotation engine enforces a hard programmatic gate: if $\text{Contribution Margin \%} < 35.0\%$, the API blocks quotation release and flags `REQUIRES_EXECUTIVE_DISPENSATION`.

---

## 4. Dual-Language Parity: TypeScript vs. Python

Modern web applications often compute pricing in two places:

1. **Frontend (TypeScript):** Live interactive sliders and instant quote feedback on the `/bom-builder` UI.
2. **Backend (Python / FastAPI):** Strict database serialization, PDF quotation generation, and ERP posting.

### The IEEE-754 Floating Point Drift Trap

JavaScript numbers are double-precision IEEE-754 floats. Python `Decimal` is fixed-point arbitrary precision.

```javascript
// JavaScript Frontend
0.1 + 0.2 === 0.30000000000000004 // TRUE
```

If an estimator quotes 100,000 units on the frontend at `$1.42857` per unit, a naive JavaScript `Number.toFixed(2)` will compute `$142,857.00`, while the Python backend computing with `Decimal('1.4285714')` outputs `$142,857.14`. This 14-cent discrepancy creates legal and invoice reconciliation failures.

### The Parity Architecture

```
[ Frontend: TypeScript Engine ] <--- Shared JSON Fixture ---> [ Backend: Python Engine ]
     (Decimal.js library)            (1,000 Unit Test Cases)         (Python decimal.Decimal)
```

1. **Use `decimal.js` or `bignumber.js` on Frontend:** Never use native JS `+` or `*` for currency calculations.
2. **Shared Parity Test Suite:** A single JSON test matrix containing 1,000 edge cases (zero volumes, extreme aspect-ratio PCBs, multi-currency conversions) is executed in both `npm test` (Jest) and `pytest`.

```typescript
// TypeScript shared calculation module (syn-standard.ts)
import { Decimal } from 'decimal.js';

export function computeGrossUpPrice(
  directCost: Decimal,
  targetMarginRate: Decimal,
  commissionRate: Decimal
): Decimal {
  const one = new Decimal(1);
  const basePrice = directCost.dividedBy(one.minus(targetMarginRate));
  const finalPrice = basePrice.dividedBy(one.minus(commissionRate));
  return finalPrice.toDecimalPlaces(4, Decimal.ROUND_HALF_UP);
}
```

---

## Production Hardening Checklist

```
[ ] 1. NEVER USE NATIVE FLOATS FOR CURRENCY
       Enforce Decimal types across both frontend (decimal.js) and
       backend (Python Decimal / SQL NUMERIC(15,4)).

[ ] 2. ENFORCE GROSS-UP DIVISION OVER ADDITIVE MARKUP
       Formula must always be Cost / (1 - Margin), never Cost * (1 + Margin).

[ ] 3. ISOLATE MATERIAL OVERHEAD FROM LABOR
       Apply import freight and tariffs strictly to raw materials, not to
       assembly or testing labor lines.

[ ] 4. HARD-CODE AQL 0.65 STATISTICAL BRACKETS
       Do not permit manual guessing of QA sample sizes; map lot size to
       inspection sample hours deterministically.

[ ] 5. AUTOMATED DUAL-ENGINE PARITY TESTS
       Run identical JSON quotation test suites against both TypeScript
       and Python engines in CI pipelines to prevent client-server drift.
```

---

## Related Posts

- {% post_url 2026-05-27-pcba-rfq-costing-pattern %} &#8212; Baseline cost breakdown structures for printed circuit assembly.
- {% post_url 2026-06-29-gerber-pcb-costing %} &#8212; Extracting layer counts, copper weight, and surface finish metrics from Gerber files.
- {% post_url 2026-06-29-orthogonal-state-machines %} &#8212; Modeling multi-department quotation approval workflows.
- {% post_url 2026-08-26-dual-ledger-drift-and-snapshot-reconciler %} &#8212; Aligning operational inventory counts with financial cost bases.
