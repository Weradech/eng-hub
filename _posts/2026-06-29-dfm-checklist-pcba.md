---
title: "DFM Checklist for PCBA: What SMT and Wave Solder Actually Need"
date: 2026-06-29 10:10:00 +0700
categories: [NPI, DFM]
tags: [dfm, dfa, pcba, smt, pcb, npi, manufacturing]
---

> **TL;DR** — DFM is the cheapest quality gate in your NPI process. Every item on this checklist costs ฿0 to fix at EVT layout and ฿10,000–฿100,000 to fix in production. The list below is derived from real assembly failures, not textbook theory.

---

![DFM is the cheapest quality gate in NPI: a layout issue costs nothing to fix at EVT, some rework at prototype, and 10,000 to 100,000 baht in production. The review checks SMT placement, reflow solder, wave solder, pick and place, and test access]({{ "/assets/img/2026-06-29/dfm-checklist-pcba.svg" | relative_url }})
_The same fix is free on the screen and a fortune on the line — DFM is the one gate that costs nothing to pass._

## Why DFM Gets Skipped (and Why That's Expensive)

DFM review happens at the worst possible time: after the schematic is finalised and the PCB layout is "done," but before the first boards are ordered. Engineers are tired, the schedule is slipping, and the board "looks fine" on screen.

The cost asymmetry is brutal:

| Stage | Cost to Fix a DFM Issue |
|-------|------------------------|
| Layout (pre-fab) | ฿0 — Gerber edit |
| After first board (EVT) | ฿3,000–฿15,000 — re-spin fab + assembly |
| After DVT / pre-production | ฿30,000–฿100,000 — re-spin + re-validation |
| Production run | ฿100,000+ — scrap, rework, line stoppage |

A 30-minute DFM review at EVT is the single highest-ROI activity in NPI.

---

## SMT Placement Constraints

**Board edge clearance:**  
Keep all components ≥3mm from the board edge. Conveyor rails grip the board along the edge; components too close get clipped, tombstoned, or physically broken during transport. On Aluminium PCBs, this is critical because the board is often manually fixtured — allow ≥5mm.

**Component-to-component clearance:**  
Minimum 0.15mm between courtyard boundaries in the EDA tool. In practice, use 0.25mm for SMD passives and 0.5mm for ICs with exposed pads. Courtyard violations cause placement head collisions on high-speed pick-and-place machines.

**Tall component placement:**  
Components >8mm tall (electrolytic caps, through-hole headers, large inductors) create a shadow zone for the reflow oven's convection. Place tall components at the trailing edge of the board (last through reflow) or separate them to a wave-solder pass. Mixing 15mm caps with 0402 resistors under the same reflow profile creates cold joints.

**Polarity markers:**  
Every polarised component — capacitors, diodes, LEDs, ICs — must have its polarity marker visible after placement. If the silkscreen is under the component body, operators cannot verify orientation during inspection. The "1" pin marker or cathode bar must be in the courtyard or on the adjacent silkscreen area.

---

## Reflow Solder Concerns

**Tombstoning on 0402 and smaller:**  
Tombstoning (one end lifts during reflow) happens when the land pattern is asymmetric — one pad heats faster than the other. The solder on the hot pad melts and pulls the component vertical. Root causes: unequal pad sizes, one pad near a thermal mass (ground plane), or asymmetric solder paste deposit. Use IPC-7351 standard land patterns. For 0201 and 01005, paste aperture size is critical — reduce aperture area by 10–15% relative to pad size.

**Via-in-pad:**  
Placing a via inside an SMD pad (especially QFN, BGA) causes solder to wick into the via barrel during reflow, leaving a starved joint with insufficient solder volume. If via-in-pad is unavoidable for routing, the via must be filled and capped with copper (epoxy-filled + electrolytic copper cap). Unfilled via-in-pad = guaranteed cold joint on QFN exposed pads.

**Thermal relief on GND planes:**  
ICs with ground pads connected directly to a solid copper plane dissipate heat so fast that the joint never reaches liquidus temperature. All pads connecting to internal ground planes need thermal relief spokes (4-spoke, 0.3mm spoke width minimum for standard SMD, disable thermal relief for high-current paths > 3A).

---

## Wave Solder (THU) Constraints

**Component orientation:**  
Through-hole components on the wave solder side must be oriented so their leads enter the wave perpendicular to the wave direction, not parallel. Parallel orientation creates solder bridging between adjacent leads — especially on DIP ICs and connector headers.

**SMD components on the bottom side:**  
If SMD components are glued to the bottom side for wave soldering (Type 2C assembly), the component must be within the approved wave-solder-compatible footprint list. Large SMD components act as wave deflectors and create shadow zones on downstream through-hole leads. QFN, BGA, and fine-pitch ICs must never be on the wave-solder side.

**Keep-out from fiducials:**  
5mm clear radius around each fiducial marker. Vision systems use fiducials for board registration; flux residue or a stray component near a fiducial causes misreads and mis-registration.

---

## Pick and Place Requirements

**Fiducial count and placement:**  
- Minimum 2 fiducials per board (3 preferred for panelised boards)
- Minimum 3 fiducials per panel
- Fiducials must be in opposite corners — never aligned (collinear fiducials give ambiguous rotation)
- Fiducial pad: 1.0–1.5mm copper circle, no soldermask, 3mm clearance from nearest component

**Nozzle access:**  
Pick-and-place nozzles need clearance above the target component. A tall component (e.g., a 10mm inductor) within 2mm of a 0.5mm pitch QFP blocks the nozzle approach path. The machine either skips the component or crashes. Review 3D clearance, not just top-view courtyard.

**Package compatibility:**  
Confirm that every component in the BOM has a pick-and-place compatible package. Odd-form components (press-fit connectors, battery holders, RF shields) need manual placement — note these explicitly so line operators don't attempt P&P on them.

---

## Test Access

**ICT test points:**  
- Minimum pad size: 1.0mm diameter (1.5mm preferred for bed-of-nails)
- Test point pitch: ≥2.54mm center-to-center (standard probe pitch)
- Test point coverage target: ≥80% of net nodes accessible from bottom side
- Never place test points under BGA or QFN packages — they are inaccessible to probes

**FCT connector accessibility:**  
FCT connectors (debug UART, programming header, power input for test) must be accessible with the board mounted in the FCT fixture. If the connector is on the underside and the fixture blocks it, FCT requires a custom nest or a manual cable attachment — both add cycle time.

---

## DFM Checklist (Compact Reference)

| # | Item | Pass Criterion | Common Failure |
|---|------|---------------|----------------|
| 1 | Board edge clearance | ≥3mm all components | Connector too close to edge |
| 2 | Component courtyard clearance | ≥0.15mm between courtyards | ICs placed tight for routing |
| 3 | Tall component placement | Trailing edge or separate pass | Tall cap shadows 0402s |
| 4 | Polarity markers visible | Marker outside component body | Marker under IC package |
| 5 | Land pattern standard | IPC-7351 / IPC-SM-782 | Custom pads with wrong ratio |
| 6 | Via-in-pad | Filled + capped, or no via | Open via under QFN pad |
| 7 | Thermal relief on GND | 4-spoke ≥0.3mm for SMD | Solid plane, cold joint |
| 8 | THU orientation vs wave | Perpendicular to wave direction | DIP parallel → bridge |
| 9 | Bottom-side SMD for wave | Approved wave-compatible list | BGA on wave side |
| 10 | Fiducials — count | ≥2 per board, ≥3 per panel | 1 fiducial, ambiguous rotation |
| 11 | Fiducials — clearance | ≥5mm from nearest component | Component in clear zone |
| 12 | Fiducials — no soldermask | Bare copper, no mask | Mask over fiducial |
| 13 | Fiducials — not collinear | Corner positions | Both fiducials on same axis |
| 14 | ICT test point size | ≥1.0mm pad | 0.6mm pad, probe slips |
| 15 | ICT test point coverage | ≥80% nodes accessible | Major nets with no TP |
| 16 | ICT probe pitch | ≥2.54mm center-to-center | Dense TPs, probes collide |
| 17 | FCT connector access | Accessible in fixture | Connector blocked by fixture |
| 18 | Solder paste aperture (0402–) | 90% of pad area | 100% → bridging |
| 19 | QFN/BGA thermal via | Via filled if in pad | Wick → starved joint |
| 20 | P&P nozzle clearance | 3D clearance check | Tall cap blocks nozzle approach |

---

## When to Do the DFM Review

The trigger is **before the first PCB fabrication order**, which means:

```
NPI Stage:   PoC → EVT → DVT → PVT → MP
DFM Review:       ↑
              After layout, before Gerber release to fab
```

By EVT, the schematic is stable enough that the layout is representative. After DVT, a DFM finding means a re-spin — expensive. The DFM review at EVT is the last cheap opportunity.

In practice: when the engineer says "layout is done, ready to order boards" — that is the DFM review trigger. Block the Gerber release, do the 30-minute review with the checklist, then release. The delay is 30 minutes. The alternative is a 2-week re-spin cycle.
