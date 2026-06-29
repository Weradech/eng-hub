---
title: "Sizing a PCB from Gerber Files: FR-4 vs Aluminium Costing"
date: 2026-06-29 10:00:00 +0700
categories: [NPI, PCB]
tags: [pcb, gerber, costing, rfq, npi, eda]
---

> **TL;DR** — Parse the outline/mechanical layer (_G1, .GKO, .GM1) to extract the board bounding box, compute area, apply panel yield, and divide the panel price by boards-per-panel. FR-4 and aluminium differ by 2–3× on substrate cost and have different stack-up rules that change the identification method. Getting this wrong at RFQ stage means a 2.5× pricing error before you've quoted a single component.

---

![Sizing a PCB from Gerber files: parse the outline layer, extract the bounding box, compute area, apply panel yield, then divide panel price by boards-per-panel for unit cost. FR-4 and aluminium differ 2 to 3 times on substrate and are identified differently. Quoting from assumed size instead of the parsed outline caused a 2.5 times error]({{ "/assets/img/2026-06-29/gerber-pcb-costing.svg" | relative_url }})
_Size the board from the outline layer, not the eyeball — every downstream cost multiplies that one number._

## Which Layer Is the Outline

Gerber file naming conventions are set by the EDA tool, not a universal standard. The mechanical/outline layer has many aliases:

| EDA Tool | Outline Layer Filename |
|----------|----------------------|
| Altium Designer | `*.GM1`, `*.GKO`, `*_Board_Outline.gbr` |
| KiCad | `*-Edge_Cuts.gbr`, `*.GKO` |
| Eagle | `*.GML` |
| PADS | `*_G1.gbr`, `*_outline.gbr` |
| Generic RS-274X | `*.GBR` with layer comment `G1` or `OUTLINE` |

The quickest identification method: open the Gerber in any viewer and look for a single closed polygon with no fills, pads, or traces — just the board perimeter. In RS-274X format, the outline is drawn with D01 (draw) or D02 (move) aperture commands. The bounding box is the min/max X and Y coordinates across all D01 commands on that layer.

```
# Pseudo-code: extract bbox from outline Gerber
x_coords = [coord.x for cmd in gerber.outline_layer if cmd.type == 'D01']
y_coords = [coord.y for cmd in gerber.outline_layer if cmd.type == 'D01']
width_mm  = (max(x_coords) - min(x_coords)) * unit_scale
height_mm = (max(y_coords) - min(y_coords)) * unit_scale
area_dm2  = (width_mm * height_mm) / 10000
```

Units in RS-274X are specified in the file header (`%MOIN*%` for inches, `%MOMM*%` for mm). Most modern EDA tools default to mm.

---

## FR-4 vs Aluminium: How to Tell From the Gerber Package

This is the identification that trips up RFQ engineers most often. The substrate type is not stated anywhere in the Gerber files — you infer it from the layer stack.

**FR-4 indicators:**
- Multiple copper layers (Top, Bottom, + inner layers for 4L/6L)
- Via drill file present (`.DRL`, `.XLN`, `.TXT`) — FR-4 is drillable
- PTH (plated through-hole) components are normal

**Aluminium PCB indicators:**
- **Single copper layer only** — aluminium cannot have inner layers or through-vias
- No via drill file, or drill file contains only NPTH (unplated mounting holes)
- Often only a Top copper layer (`*.GTL`) with no Bottom copper (`*.GBL`)
- Aluminium substrate acts as both the base material and the heatsink — the dielectric layer is a thin (~75–150µm) thermally conductive coating between the copper and the aluminium

The physical rule: you cannot drill a plated via through aluminium without shorting the copper to the metal substrate. So any design requiring vias is FR-4 by definition. LED driver boards, LED bars, high-current single-layer designs → almost always aluminium.

**Quick check when opening a Gerber package:**
1. Count copper layers: 1 copper only → likely Alu
2. Check for `.DRL` file: absent or NPTH-only → confirms Alu
3. Look at board dimensions: thin elongated strips (e.g., 150×50mm LED bars) with single copper → very likely Alu

---

## Board Area Calculation and Panel Yield

Once you have the bbox, the costing model is:

```
board_area_dm2 = (width_mm × height_mm) / 10000

# A4 panel method (210 × 297mm usable = ~588 cm² ≈ 5.88 dm²)
# Subtract 5mm border all around → usable: 200 × 287mm

boards_per_panel = floor(200 / width_mm) × floor(287 / height_mm)
cost_per_board   = panel_price / boards_per_panel
```

```
A4 panel (200 × 287mm usable)
┌─────────────────────────────────┐
│  [board][board][board][board]   │  ← row 1
│  [board][board][board][board]   │  ← row 2
│  [board][board][board][board]   │  ← row 3
│                                 │
└─────────────────────────────────┘
boards_per_panel = floor(200/W) × floor(287/H)
```

Standard panel prices (Thai PCB suppliers, 2025–2026):

| Substrate | Layer | Panel Price (฿) | Notes |
|-----------|-------|----------------|-------|
| FR-4 | 2L, 1.6mm | 180–220 | Standard lead time 5–7 days |
| FR-4 | 4L, 1.6mm | 450–600 | Lead time 7–10 days |
| Aluminium | 1L, 1.6mm | 400–700 | IMS (Insulated Metal Substrate) |
| Aluminium | 1L, 3.0mm | 600–900 | High-power, thicker heatsink |

Area-based pricing (฿/dm²) is used when boards don't pack well into A4:

| Substrate | Layer | ฿/dm² |
|-----------|-------|-------|
| FR-4 2L | Standard | 80–120 |
| FR-4 4L | Standard | 200–350 |
| Aluminium 1L | IMS | 200–350 |
| Aluminium 1L | High-power | 300–500 |

Use whichever method gives the lower cost — suppliers quote both. For small boards that pack well (>6 per A4), the panel method is almost always cheaper.

---

## The K3VG Lesson: A 2.5× Costing Error

A headlamp LED bar project arrived with a Gerber package labelled only `K3VG_PCB_Rev01`. The package contained:
- `K3VG_GTL.gbr` — Top copper
- `K3VG_G1.gbr` — Outline
- `K3VG_GTS.gbr` — Top soldermask
- `K3VG_GTP.gbr` — Top silkscreen
- `K3VG_DRL.txt` — Drill file (NPTH only, 4× M3 mounting holes)

No bottom copper, no inner layers, no plated vias. The drill file had only 4 NPTH holes — mounting only.

Parsing `K3VG_G1.gbr`: board size **150 × 50mm** (elongated LED strip).

```
boards_per_panel = floor(200/150) × floor(287/50)
               = 1 × 5 = 5 boards per A4 panel

FR-4 2L panel:  ฿200 / 5 = ฿40/board  ← wrong substrate assumption
Alu IMS panel:  ฿500 / 5 = ฿100/board ← correct
```

The costing error: **฿40 vs ฿100 per board = 2.5× underquote on substrate alone**. On a 500-unit run, that's ฿30,000 in substrate cost not captured at RFQ.

The tell that should have triggered the check: a single copper layer on a narrow LED strip driving high-current LEDs is the canonical aluminium application. Copper-only Gerber package + NPTH-only drill = Alu until proven otherwise.

---

## Practical Workflow at NPI Stage

1. **Receive Gerber package** → count copper layers first
2. **If 1 copper layer** → default to Aluminium assumption, confirm with customer or EDA BOM note
3. **Parse outline layer** → extract bbox in mm
4. **Run both methods** (A4 panel + area) → take lower
5. **Quote at correct substrate price** → Alu is 2–3× FR-4, never blend them
6. **Note in RFQ** the substrate assumption — if customer corrects it, the price changes

A Gerber-to-cost Python script that does steps 1–4 in under 5 seconds eliminates this class of error entirely. The investment is 2–3 hours of scripting; the payoff is every future RFQ.
