---
title: "ICT Methodology Deep Dive — Test Types, Fixtures, Coverage, and Decision Framework"
date: 2026-05-29 14:00:00 +0700
categories: [NPI, Test]
tags: [ict, test-engineering, pcba, dft, fixture, bed-of-nails, flying-probe, jtag, boundary-scan]
---

> **TL;DR** — Select ICT from the faults the product needs to detect, physical access, lot size and the actual fixture/program quote. The numerical examples on this page are unverified planning inputs, not universal costs, coverage or acceptance thresholds.

![Illustrative comparison of bed-of-nails and flying-probe ICT by fixture investment, test time and product-specific fault coverage]({{ "/assets/img/2026-05-29/ict-methodology-deep-dive.svg" | relative_url }})
_Bed-of-Nails or Flying Probe is a volume call — pay the fixture once, or pay per board in time._

This is a companion to the [ICT Assessment Pattern]({% post_url 2026-05-27-ict-assessment-pattern %}) post — that post covers the *assessment workflow*; this post covers the *engineering knowledge* needed to make decisions inside that workflow.

**Editorial caution (2026-09-29):** The fixture prices, cycle times, geometric rules and volume thresholds below lack a linked current supplier quote or product-specific validation. Treat them only as prompts for questions, not as procurement, DFx or release limits. Establish product coverage with a [requirements-to-verification matrix]({% post_url 2026-09-29-requirements-to-verification-traceability %}) and actual test evidence.

---

## What ICT Actually Tests

ICT (In-Circuit Test) is a **component-level electrical test** performed on a populated PCB. Probes contact dedicated test points on each net, then measure electrical properties to confirm every component:

- Is the **right part** (correct value/part number)
- Is **placed correctly** (right footprint, right side)
- Has the **correct polarity** (diodes, electrolytic capacitors, BJTs)
- Is **electrically connected** (no opens, no shorts, no cold joints)

**Critical limitation:** ICT does *not* verify function. A board can pass ICT and still fail to operate because of firmware bugs, timing issues, or marginal performance. That's why ICT is always paired with FCT downstream.

```
SMT/THU → AOI (visual) → ICT (electrical) → FCT (functional) → Box Build → Final QC
```

---

## Test Types — What ICT Measures

### 1. Analog Parametric (Passive Components)

| Component | Method | Typical Range |
|-----------|--------|---------------|
| Resistor | 4-wire Kelvin (eliminates probe resistance) | 0.1 Ω – 10 MΩ |
| Inductor | AC stimulus + phase measurement | 1 µH – 1 H |
| Capacitor | AC bridge or charge-discharge | 1 pF – 10 mF |
| Diode | Forward voltage (V_f) | Si ~0.6 V · Schottky ~0.3 V · LED 1.8–3.6 V |
| Zener | Reverse breakdown voltage | per spec |
| BJT | h_FE + polarity | NPN/PNP detection |
| MOSFET | V_GS(th) + polarity | per spec |
| Fuse | Continuity (~0 Ω) | Pass = 0 |

### 2. Shorts and Opens

- **Continuity sweep** — paired probe test across all node pairs to find unintended shorts
- **Solder bridge detection** — finds adjacent-pin shorts
- **Lifted pin / dry joint** — open between IC pin and net
- **Trace damage** — open in long traces

### 3. Digital / IC Verification

| Method | What it does | Best for |
|--------|--------------|----------|
| **Boundary Scan (JTAG, IEEE 1149.1)** | Shifts pattern through IC boundary register and reads back | Verifying ICs are present and pins connected |
| **IC Vector Test** | Sends input pattern, reads expected output | Logic IC presence verification |
| **Backdriving** (legacy) | Forces pin states through neighbouring drivers | Avoid in modern ICT — risk of latch-up/ESD |

### 4. Power-On Safety Check

- **V_cc rail verification** at low current limit (3.3 V / 5 V / 12 V within tolerance)
- **Inrush current limit** — abort if exceeds threshold (likely short circuit)

---

## Fixture Types — Selection Matrix

### A. Bed-of-Nails (BoN)

Spring-loaded pogo pins arranged on a fixture plate. The board is pressed down (vacuum or mechanical clamp), making all probe contacts simultaneously.

| Parameter | Value |
|-----------|-------|
| Pin density | 50 mil (1.27 mm) standard · 39 mil (1 mm) tight · 100 mil (2.54 mm) legacy |
| Pin types | Spring (0.5–1.5 A) · High-current (3–5 A) · Kelvin pair (4-wire) |
| NRE | ฿80K – ฿500K+ depending on pin count |
| Cycle time | 5–30 sec/board |
| Best for | HVLM, >1,000 pcs/year, stable design |
| **Pros** | Fast, repeatable, high coverage |
| **Cons** | High NRE, design changes require new fixture |

### B. Flying Probe

One to eight movable probes that scan X-Y across the board. No fixture required.

| Parameter | Value |
|-----------|-------|
| NRE | ฿0 fixture + ฿15K–40K program development |
| Cycle time | 2–10 min/board (30–60× slower than BoN) |
| Best for | LVHM, prototype, NPI samples, <500 pcs/year |
| **Pros** | No fixture cost, fast design change turnaround, access any pad |
| **Cons** | Very slow throughput, not suitable for mass production |
| Vendors | Takaya · SPEA · Seica · Acculogic |

### C. Clamshell (Dual-Sided BoN)

BoN that presses both top and bottom simultaneously. Used when components/TPs require dual-side access.

- Cost: 1.5–2× single-side BoN
- Best for: dual-side critical access (most digital boards can avoid this with through-vias)

### D. In-Line ICT (High-Speed Production)

Conveyor-mounted ICT in SMT line with automated load/unload.

- Cycle time: <10 sec/board
- Best for: >10K pcs/month volume
- High capital investment

---

## DFT (Design for Test) — 9-Point Checklist

Use during schematic and PCB review **before** committing to ICT. Catches design issues while changes are still cheap.

| # | Criterion | Pass Condition |
|---|-----------|----------------|
| 1 | Board size | Within fixture frame (standard 250 × 300 mm) |
| 2 | Pin budget | TP count ≤ fixture capacity (128/256/512 pin standard) |
| 3 | Via accessibility | Through-vias accessible from one side preferred |
| 4 | Passive value range | R/L/C within measurable range (see table above) |
| 5 | Diode/LED V_f specified | V_f values declared in test file |
| 6 | Boundary scan chain | JTAG TAP accessible, chain documented |
| 7 | Double-side requirement | Single-side preferred (avoids clamshell cost) |
| 8 | Custom/encrypted IC | Bypass plan or move to functional test |
| 9 | TP pad size | ≥ 35 mil (0.9 mm) diameter, ≥ 50 mil center-to-center |

---

## Coverage Metrics

### Nodal Coverage (Geometric)

```
Nodal Coverage (%) = (TPs probed / Total nets) × 100
```

- ≥ 90% — Excellent
- 80–90% — Good
- < 80% — Add test points or accept coverage gap

### Fault Coverage (Test-Based)

```
Fault Coverage (%) = (Detectable faults / Total possible faults) × 100
```

There is no product-independent percentage for ICT, AOI and FCT combined. List the relevant fault classes, test accessibility and detection method for the actual board. Report observed and planned coverage separately; do not add nominal percentages from different methods.

---

## NRE Breakdown — Reference Numbers

| Item | Cost (THB) | Notes |
|------|-----------|-------|
| Fixture mechanical (BoN frame + plate) | 28,000 – 40,000 | Depends on pin count |
| Test program development | 12,000 – 18,000 | 5–8 engineer-days |
| Debug + first article validation | 3,000 – 5,000 | 1–2 days |
| Documentation (test plan + SOP) | 1,500 – 3,000 | |
| **Total** | **฿44,500 – ฿66,000** | Via-only probe can save 10–15% |

**Break-even volume:** calculate from current fixture/program cost, per-unit cost, test time, expected revisions and order pattern. The historical table above does not establish a universal 200-piece threshold.

---

## Throughput Reference

| Method | Cycle Time | Throughput |
|--------|-----------|------------|
| BoN ICT | 5–30 sec | 100–300 boards/hr |
| Flying Probe | 2–10 min | 6–30 boards/hr |
| In-Line ICT | <10 sec | 360+ boards/hr |

---

## Equipment Vendors

| Vendor | Category | Notes |
|--------|----------|-------|
| Teradyne TestStation | Mass-production BoN | High-end |
| Keysight 3070 | Industry standard | Mid–high tier |
| GenRad / Aeroflex | Classic ICT | Legacy installs |
| Takaya | Flying Probe | NPI / prototype |
| SPEA 4060 | Flying Probe | High-speed flying |
| JTAG Technologies | Boundary scan specialist | Complementary tool |
| Seica | Flying + BoN hybrid | EU market |
| KYORITSU FOCUS-2000 | Mid-range BoN | Common in TH customer base |

---

## Decision Framework

```
Volume and revision pattern?
├─ Low or uncertain volume → compare flying-probe and targeted functional-test offers
├─ Repeated stable builds → compare bed-of-nails fixture cost and per-unit saving
└─ High throughput demand → assess in-line options against actual takt time

Design lifespan?
├─ Prototype / one-off → Flying Probe
├─ 6 months – 2 years → BoN
└─ > 2 years          → BoN + fixture maintenance plan

Access requirement?
├─ Single-side TP sufficient → BoN single-side (lowest cost)
├─ Both sides required       → Clamshell (1.5–2× cost)
└─ BGA / hidden joints       → ICT + X-ray (ICT alone insufficient)
```

---

## When to Recommend Skipping ICT

- **LED-only board** — V_f can be verified during FCT
- **Single-IC simple board** — AOI + FCT provides adequate coverage
- **Low or uncertain volume** — calculate whether fixture NRE can be recovered
- **Frequent design revisions** — Fixture sunk cost too high

## When ICT Deserves Explicit Assessment

- **Mixed analog + digital** with high passive count
- **Safety-related requirements** — use the applicable customer and regulatory requirements to decide the required test evidence
- **High passive count** (>50 R/C/L) — manual probe verification impractical
- **Customer audit requirement** — component-level traceability needed

---

## Defects ICT Catches

| Defect | Detection question for this design |
|--------|------------------------------------|
| Solder bridge or open | Are both relevant nodes accessible, and will this program detect the fault? |
| Missing or wrong-value component | Can the installed network be measured without an ambiguous parallel path? |
| Wrong polarity | Does the chosen check distinguish polarity under safe conditions? |
| Lifted pin or dry joint | Is the affected connection observable by probe or another method? |
| Damaged component | Which electrical or functional symptom would the check detect? |
| Trace damage | Are the affected endpoints included in the test? |

## Coverage Gap — What Only FCT Can Catch

- Firmware bugs
- Clock/timing drift
- EMI/EMC issues
- Thermal-dependent faults
- Marginal performance (in-spec but near limits)
- Interaction defects (all components fine individually, system fails)
- Functional sequence errors

These all require functional testing — see the [FCT Methodology Deep Dive]({% post_url 2026-05-29-fct-methodology-deep-dive %}).

---

## Pricing in RFQ

- **ICT NRE** as a separate line item: obtain a current fixture and program quotation for the actual board.
- **ICT per-board test fee**: use the current cycle, labor, equipment and supplier basis.
- **Do not bundle** ICT NRE into unit price — keep it visible so the customer can evaluate volume break-even themselves

---

## Related Posts

- [ICT Assessment Pattern]({% post_url 2026-05-27-ict-assessment-pattern %}) — 5-step assessment workflow
- [FCT Methodology Deep Dive]({% post_url 2026-05-29-fct-methodology-deep-dive %}) — functional test counterpart
- [PCBA RFQ Costing Pattern]({% post_url 2026-05-27-pcba-rfq-costing-pattern %}) — how to price ICT/FCT in quotation
