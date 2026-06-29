---
title: "Going Paperless on GR→IQC: A 3-Phase Plan That Actually Works"
date: 2026-06-29 10:40:00 +0700
categories: [Process, Operations]
tags: [paperless, gr, iqc, wms, operations, digitisation, npi]
---

> **TL;DR** — Full paperless from day 1 fails because operators use paper as a cognitive anchor, not just a recording medium. A 3-phase hybrid approach — paper safety net → on-demand paper → 100% digital — lets you digitise the process without digitising the resistance. The transition takes 8–12 weeks per phase; rushing it creates more paper, not less.

---

## Why "Just Go Digital" Fails

The standard digital transformation pitch: "We'll replace the paper form with a tablet app. Same fields, better data."

What actually happens in week 2: the paper forms are still there because "sometimes the tablet is slow" or "I can't find where to enter the lot number" or "the Wi-Fi dropped and I didn't know if it saved." By week 4, you have dual systems — paper and digital — with all the overhead of both and the reliability of neither.

The root cause: **operators use paper to think, not just to record.** A paper IQC form is a working document — operators write provisional counts, cross things out, add margin notes, circle anomalies. It is a workspace, not a form. Removing the workspace before the digital equivalent works as a workspace creates errors, not efficiency.

The goal of Phase 1 is not to remove paper. It is to make the digital system load-bearing before the paper safety net is taken away.

---

## Phase 1: Hybrid (Digital Primary, Paper Backup)

**Duration:** 4–6 weeks  
**Target users:** Receiving dock (GR), QA lab (IQC)

### What changes

- Receiving scans the shipment barcode → WMS automatically creates a GR record linked to the PO
- WMS prints a GR label (barcode + PO number + date + quantity)
- The GR label is attached to the physical goods moving to IQC staging
- QA pulls up the GR record in WMS using the label barcode — the PO, supplier, and item list are already populated
- QA still uses the paper IQC form for inspection notes and measurements
- QA enters the final disposition (PASS / HOLD / REJECT) into WMS at the end

### What doesn't change

- Paper IQC form stays. QA uses it exactly as before for working notes.
- Paper is the workspace; WMS is the record of record.

### Success metric

≥80% of GR events have a WMS record at the time IQC begins (before Phase 2, this was 0% — IQC had no idea if a GR was in the system until someone checked).

### Common blockers in Phase 1

**Barcode not on the supplier label.** Some suppliers ship with no machine-readable code. Solution: a receiving WI that maps non-barcoded goods to PO manually (type the PO number). Annoying but survivable. Flag the supplier for EDI or label requirement in the next PO.

**WMS too slow at the dock.** If the GR scan takes >5 seconds to respond, operators will stop scanning. This is a system performance issue, not a training issue. Fix the latency before blaming adoption.

**IQC result not entered until end of day.** The target is real-time entry. If QA batches entries at 17:00, downstream (Stores, Procurement) sees everything as pending until EOD. Habit change, not a system change — QA supervisor must model the behaviour.

---

## Phase 2: On-Demand Paper

**Duration:** 4–6 weeks  
**Target users:** QA lab (IQC), QA supervisor

### What changes

- No pre-printed paper IQC forms. The template exists but is not printed routinely.
- QA opens WMS on a tablet or PC → the IQC record is pre-populated with item details from the GR
- QA completes the inspection fields in WMS directly — measurements, visual results, lot number, sample size, AQL outcome
- If QA needs a working space during inspection (complex items, multiple measurements), they print the WMS-generated inspection sheet on demand — fill it in, then transfer final results to WMS
- A compliance certificate or IQC summary is printed only if the customer requires a physical document

**Paper is now output, not input.**

### Success metric

QA can complete IQC for standard catalogue items (resistors, capacitors, connectors) without touching paper. Complex items (custom mechanical parts, PCBAs) may still use on-demand print.

### What makes Phase 2 work

**The IQC UI must be fast.** The number of taps to complete a standard PASS result must be ≤5. If it requires 12 taps and three screens, QA will print the form. Count the taps before you declare Phase 2 ready.

**Auto-fill from GR.** The IQC record must inherit supplier name, part number, PO number, and quantity from the GR. If QA has to retype any of these, the system has failed the human, not the other way around.

**Submit confirmation.** When QA submits an IQC result, the UI must clearly confirm it was saved. "Spinning wheel then blank screen" causes re-submits, which causes duplicate records. A green "Saved ✓" with the record number is the minimum.

---

## Phase 3: 100% Digital

**Duration:** Ongoing  
**Target users:** All stakeholders — Receiving, QA, Procurement, Stores, Management

### What changes

- Mobile-first UI on phones (360px layout, large tap targets, offline-tolerant)
- QA uses a phone: scan GR barcode → inspection record opens → tap through fields → photograph nonconforming items → submit
- Paper only for regulatory requirements (import documents, customer-mandated CoC) or exceptional nonconformance cases requiring physical sign-off
- Nonconformance photos attached to the IQC record in WMS — no separate filing, no lost paperwork

### Success metric

<5% of GR/IQC events involve paper. The exceptions are documented (which items, which customers, why).

### What breaks Phase 3

**Poor Wi-Fi at the receiving dock.** This is infrastructure, not culture. Map the signal strength before rollout. If the dock has dead zones, deploy a Wi-Fi AP or use offline-tolerant mode (local cache + sync on reconnect). Don't tell operators to "move closer to the router."

**No photo attachment on nonconformance.** Operators will revert to paper to create the evidence trail if the system can't store photos. Photo upload is not a nice-to-have in Phase 3; it is a requirement.

**QA supervisor not trained before rollout.** If the supervisor still accepts paper forms from the team (because it's faster for them to process), the team will use paper. Supervisors must be fluent in the WMS IQC module before operators are trained on it. Train top-down, not bottom-up.

**Reporting not ready.** Management needs to see IQC metrics (pass rate by supplier, defect by item category) from the WMS before they trust it. If reports still come from Excel, the system is perceived as a data entry burden, not a source of truth. Build the reports before Phase 3, not after.

---

## Field UX Requirements (Non-Negotiable)

These are not preferences. If any of these fail, Phase 3 fails.

| Requirement | Threshold | Why |
|-------------|-----------|-----|
| Page load time | <2 seconds on Wi-Fi | >3s = operator stops waiting |
| Minimum tap target | 44×44px | Human finger tip accuracy |
| Offline tolerance | Queue + sync on reconnect | Dock has intermittent Wi-Fi |
| Auto-fill from barcode scan | PO + item + quantity | Retype = error + frustration |
| Submit confirmation | Visible, with record number | Without this: re-submits |
| Photo capture | Built into IQC form | External app = lost link |
| Layout width | 360px minimum viewport | Field phones, not office monitors |

Test on a real receiving dock environment — fluorescent lighting, gloves, noisy background — before declaring the UI ready. An office demo is not a field test.

---

## The Timeline in Practice

```
Week 1–2:   Deploy GR barcode scan in WMS, train dock (Phase 1)
Week 3–4:   Monitor adoption, fix latency and edge cases
Week 5–6:   Train QA on WMS IQC entry (Phase 1 complete target: 80%)
Week 7–8:   Remove pre-printed forms, switch to on-demand (Phase 2 begins)
Week 9–10:  Tune IQC UI tap count, fix submit feedback
Week 11–12: Phase 2 stable — measure: can standard items complete without paper?
Week 13+:   Mobile-first rollout, photo attachment, Phase 3 begins
Month 4–5:  Full Phase 3, paper <5% of events
```

Skipping phases doesn't compress the timeline. It compresses the phase and then extends the backslide. Operators who lost the paper safety net before the digital system was reliable will not give the system a second chance easily.
