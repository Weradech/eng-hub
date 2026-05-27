---
title: "ICT Assessment Pattern — Evaluate In-Circuit Test from NPI Stage"
date: 2026-05-27 12:00:00 +0700
categories: [NPI, Test]
tags: [ict, dft, test-engineering, npi, pcba, nre]
---

> **TL;DR** — 5 steps: receive files → count TP/via → evaluate 9 DFT criteria → estimate NRE → generate SOP report with proposal for the customer.

---

## Why Assess ICT at NPI Stage?

An ICT fixture is built once and used throughout production. If the layout doesn't support DFT:
- Fixture redesign = additional NRE of ฿50,000–500,000
- Layout ECR = 2–4 week delay
- Low coverage = defects escaping to the customer

**Golden rule: Assess ICT while layout changes are still free.**

---

## 5-Step ICT Assessment Process

### Step 1 — Receive and Review Files

Required files from customer or design team:

| File | Format | Purpose |
|------|--------|---------|
| Gerber / ODB++ | .gbr / .tgz | Count TP, via, pad |
| BOM | Excel / CSV | Map components to testability |
| Schematic | PDF / Altium | Understand net topology |
| Assembly drawing | PDF | Check component side, keep-outs |

---

### Step 2 — Count Test Points and Vias

**Count from Gerber layers:**

```
Top TPs    = Top Copper pads with drills
Bottom TPs = Bottom Copper pads
Accessible vias = Vias not obstructed by components (top view)
```

**Coverage estimate:**
```
Node count  = number of nets requiring measurement
TP count    = number of physical test points
Coverage (%) = (TP count / Node count) × 100
```

Target: **≥ 85% node coverage** for production boards.

---

### Step 3 — DFT Checklist (9 Criteria)

| # | DFT Criterion | Pass Condition |
|---|--------------|----------------|
| 1 | Test point diameter | ≥ 1.0 mm |
| 2 | Test point pitch | ≥ 2.54 mm center-to-center |
| 3 | TP clearance from components | ≥ 1.5 mm |
| 4 | TP clearance from board edge | ≥ 3.0 mm |
| 5 | Bottom-side TP ratio | ≥ 60% of all TPs (single-side fixture) |
| 6 | Vias used as TPs | No tented vias |
| 7 | Power / Ground access | TP on every main supply rail |
| 8 | Crystal / Oscillator | Disable point or bypass available |
| 9 | Programming header | JTAG/SWD/ISP pad accessible |

**Scoring:**
- 9/9 Pass → Full ICT capable
- 6–8 Pass → ICT possible with limited coverage
- < 6 Pass → Recommend FVT or AOI instead + raise ECR

---

### Step 4 — NRE Estimation

```
Fixture NRE = Base cost + (TP count × rate) + Engineering hours

Example:
Base cost          = ฿25,000
120 TPs × ฿200    = ฿24,000
Engineering 16h    = ฿12,800
──────────────────────────────
Total NRE          = ฿61,800
```

**Lead time:** 4–6 weeks after final Gerber approval.

---

### Step 5 — Assessment Report

The report should include:
1. **Board overview** — dimensions, layer count, component count
2. **TP Summary** — top/bottom count, coverage %
3. **DFT Score** — 9 criteria pass/fail with recommendations
4. **NRE Estimate** — itemized
5. **Recommendation** — ICT / FVT / AOI / combined strategy

---

## Case Study — MIRA R3

| Parameter | Value |
|-----------|-------|
| Board size | 120 × 80 mm, 4-layer |
| Component count | 187 |
| TP count (bottom) | 94 |
| Accessible vias | 38 |
| Node count | 118 |
| Coverage | 112 / 118 = **94.9%** ✅ |
| DFT Score | 8/9 (crystal disable point missing) |
| Recommendation | ICT viable + add crystal bypass resistor DNI |
| NRE Estimate | ฿68,500 |

---

## Anti-patterns

| ❌ Avoid | ✅ Instead |
|---------|-----------|
| Assess from BOM only | Always review actual Gerber |
| Count only pads labeled "TP" | Include accessible vias |
| Quote NRE as a lump sum | Itemize so customer can evaluate |
| Ignore tented vias | Every tented via = unusable as TP |
