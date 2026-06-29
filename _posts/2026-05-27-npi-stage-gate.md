---
title: "NPI Stage Gate — Why Every Gate from PoC to MP Exists"
date: 2026-05-27 13:00:00 +0700
categories: [NPI, Process]
tags: [npi, stage-gate, evt, dvt, pvt, product-development]
---

> **TL;DR** — Stage gates are not bureaucracy. Each gate is a risk firewall that prevents exponentially more expensive problems downstream.

---

![NPI stage-gate pipeline from PoC to Mass Production: each gate is a risk firewall answering one question, and the cost to fix grows tenfold per stage]({{ "/assets/img/2026-05-27/npi-stage-gate.svg" | relative_url }})
_Every gate is a risk firewall answering one question — and the cost to fix grows tenfold past each one._

## Why Stage Gates Exist

The cost to fix a problem grows exponentially with project stage:

```
Design stage:   ฿1,000      (edit schematic)
EVT stage:      ฿10,000     (layout respin)
DVT stage:      ฿100,000    (tooling rework)
Production:     ฿1,000,000+ (recall, rework, warranty claims)
```

**A gate = a decision point: are we ready to invest in the next stage?**

---

## Stage 1 — PoC (Proof of Concept)

**Goal:** Prove the core concept works in principle.

**Deliverables:**
- Breadboard / dev-kit prototype
- Key function demo (full feature set not required)
- Draft BOM (no costing needed yet)
- Go / No-Go decision

**Exit gate:** Core function demonstrated → invest in real layout

---

## Stage 2 — EVT (Engineering Validation Test)

**Goal:** Validate that the design meets every spec.

**Deliverables:**
- First PCB spin (hand-built or proto house)
- Complete schematic + layout
- BOM with approved vendors
- Test report vs. specification
- Failure analysis for every failing item

**Exit gate:** All critical specs pass → invest in DVT tooling

**Common EVT failure modes:**
- Power integrity (noise, ripple out of spec)
- Thermal violation (component exceeds Tjmax)
- EMI pre-scan failure
- Firmware instability

---

## Stage 3 — DVT (Design Validation Test)

**Goal:** Validate that the design AND process are production-ready and pass reliability.

**Deliverables:**
- Production-intent PCB (from production vendor, not prototype house)
- Tooling / fixtures / jigs complete
- Reliability tests: thermal cycling, humidity, vibration
- Regulatory pre-compliance: CE / FCC / TIS
- DFM/DFA sign-off
- SOP draft

**Exit gate:** Reliability pass + yield ≥ target → invest in PVT

**DVT is the most expensive stage to fail** because tooling costs have already been committed.

---

## Stage 4 — PVT (Production Validation Test)

**Goal:** Validate that the production line can hit volume at quality targets.

**Deliverables:**
- Pilot run of 50–500 units (depending on volume)
- Yield data from production line
- Test coverage verification (ICT/FCT pass rates)
- Packaging and label verification
- Golden sample + limit samples established

**Exit gate:** Yield ≥ target, zero critical defects → full MP ramp

---

## Stage 5 — MP (Mass Production)

**Goal:** Consistently produce at volume within cost targets.

**Ongoing activities:**
- SPC monitoring (Cpk ≥ 1.33)
- ECR process for continuous improvement
- End-of-life planning

---

## Gate Criteria Template

| Gate | Must Pass | Should Pass | Monitor |
|------|-----------|-------------|---------|
| PoC → EVT | Core function | Performance ballpark | BOM cost |
| EVT → DVT | All specs | Yield > 70% | Regulatory |
| DVT → PVT | Reliability | Yield > 90% | Cost target |
| PVT → MP | Yield ≥ target | Zero critical defects | Supplier readiness |

---

## Common Mistakes

| Mistake | Impact |
|---------|--------|
| Skipping EVT to rush DVT | DVT failure costs more + longer delay than EVT would have |
| Running DVT with prototype PCB (not production) | PVT yield diverges significantly from DVT data |
| No DFM review before DVT | Tooling rework = additional cost |
| Passing a gate with "majority pass" | Minority failures become field failures |

---

## NPI Timeline Reference

```
PoC:  2–4 weeks
EVT:  4–8 weeks  (includes 2–3 week PCB lead time)
DVT:  8–12 weeks (includes reliability testing)
PVT:  4–6 weeks
MP:   ongoing
─────────────────────────────────────────
Total NPI: 18–30 weeks (4.5–7.5 months) for medium-complexity products
```
