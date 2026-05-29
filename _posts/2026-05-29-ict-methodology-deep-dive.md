---
title: "ICT Methodology Deep Dive — Test Types, Fixtures, Coverage, and Decision Framework"
date: 2026-05-29 14:00:00 +0700
categories: [NPI, Test]
tags: [ict, test-engineering, pcba, dft, fixture, bed-of-nails, flying-probe, jtag, boundary-scan]
---

> **TL;DR** — ICT verifies *components are present and correct* (not function). Choose between Bed-of-Nails (HVLM, fast, expensive NRE) and Flying Probe (LVHM, slow, zero NRE). Typical fixture NRE ฿44.5K–66K; break-even ≥200 pcs/year. Coverage: 70–85% alone, 98%+ when combined with AOI and FCT.

This is a companion to the [ICT Assessment Pattern]({% post_url 2026-05-27-ict-assessment-pattern %}) post — that post covers the *assessment workflow*; this post covers the *engineering knowledge* needed to make decisions inside that workflow.

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

Industry benchmarks:

| Test stack | Defect coverage |
|------------|-----------------|
| ICT alone | 70–85% |
| ICT + AOI | 90–95% |
| ICT + AOI + FCT | 98%+ |
| ICT + AOI + X-ray (BGA) + FCT | 99%+ |

---

## NRE Breakdown — Reference Numbers

| Item | Cost (THB) | Notes |
|------|-----------|-------|
| Fixture mechanical (BoN frame + plate) | 28,000 – 40,000 | Depends on pin count |
| Test program development | 12,000 – 18,000 | 5–8 engineer-days |
| Debug + first article validation | 3,000 – 5,000 | 1–2 days |
| Documentation (test plan + SOP) | 1,500 – 3,000 | |
| **Total** | **฿44,500 – ฿66,000** | Via-only probe can save 10–15% |

**Break-even volume:** ≥ 200 pcs/year. Below this, Flying Probe or FCT-only is more economical.

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
Volume per year?
├─ < 200 pcs       → Flying Probe or FCT-only (ICT NRE not viable)
├─ 200–1,000 pcs   → BoN single-side
├─ 1,000–10,000 pcs → BoN single-side + spare fixture
└─ > 10,000 pcs    → In-line ICT

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
- **Volume < 200 pcs/year** — NRE doesn't break even
- **Frequent design revisions** — Fixture sunk cost too high

## When ICT is Mandatory

- **Mixed analog + digital** with high passive count
- **Safety-critical applications** — medical, automotive (ISO 26262)
- **High passive count** (>50 R/C/L) — manual probe verification impractical
- **Customer audit requirement** — component-level traceability needed

---

## Defects ICT Catches

| Defect | % of total PCBA defects | ICT detects? |
|--------|------------------------|--------------|
| Solder bridge | 20–30% | ✅ |
| Missing component | 10–15% | ✅ |
| Wrong value | 10–15% | ✅ |
| Wrong polarity | 5–10% | ✅ |
| Lifted pin | 5–10% | ✅ |
| Cold/dry joint | 10–15% | ✅ |
| Damaged component | ~5% | ✅ (partial) |
| Trace damage | < 5% | ✅ |

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

- **ICT NRE** as separate line item: ฿44.5K – ฿66K (template baseline)
- **ICT per-board test fee**: ฿15–30/pc (contract testing)
- **Do not bundle** ICT NRE into unit price — keep it visible so the customer can evaluate volume break-even themselves

---

## Related Posts

- [ICT Assessment Pattern]({% post_url 2026-05-27-ict-assessment-pattern %}) — 5-step assessment workflow
- [FCT Methodology Deep Dive]({% post_url 2026-05-29-fct-methodology-deep-dive %}) — functional test counterpart
- [PCBA RFQ Costing Pattern]({% post_url 2026-05-27-pcba-rfq-costing-pattern %}) — how to price ICT/FCT in quotation
