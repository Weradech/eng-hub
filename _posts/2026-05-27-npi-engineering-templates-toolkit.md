---
title: "NPI Engineering Templates — Copy-Paste Toolkit"
date: 2026-05-27 20:00:00 +0700
categories: [NPI, Process]
tags: [npi, templates, rca, stage-gate, sop, costing]
---

> **TL;DR** — This page contains starting layouts for an RFQ cost sheet, RCA record, stage-gate checklist and SOP. Fill in current source evidence and the authority for each decision before use.

**Editorial update (2026-09-30):** The RFQ snippet has no default overhead, margin or selling-price rule. Define the scope with the [RFQ intake record]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}) and use current offers and the organization's approved cost and commercial policy.

---

![Four copy-paste NPI engineering templates and the stage each one serves: RFQ costing, 8D/5-Why RCA, stage-gate checklist, and SOP skeleton](/assets/img/2026-05-27/npi-engineering-templates-toolkit.svg)
_Four copy-paste templates, one for each stage where engineering work tends to go undocumented._

This is a companion page, not a tutorial. Each template links back to the deep-dive that explains it:

- Costing → [PCBA RFQ Costing Pattern]({% post_url 2026-05-27-pcba-rfq-costing-pattern %})
- Stage gates → [NPI Stage Gate]({% post_url 2026-05-27-npi-stage-gate %})
- SOPs → [SOP Revamp Playbook]({% post_url 2026-05-27-sop-revamp-playbook %})
- RCA → [Buck LED Driver RCA]({% post_url 2026-05-27-buck-led-driver-rca %}) · [RCA for Software]({% post_url 2026-05-27-rca-for-software-symptoms-before-hypothesis %})

---

## 1 — RFQ costing sheet (internal working copy)

Copy into a spreadsheet as an internal working layout. The [costing method]({% post_url 2026-05-27-pcba-rfq-costing-pattern %}) explains how to establish each input and who decides what appears in the customer-facing quotation.

```csv
Section,Line item,Qty,Unit cost,Ext cost,Notes
Material,<part no / desc>,,,=Qty*UnitCost,"MOQ-adjusted price; remove DNI; PCB = own line"
Material,<part no / desc>,,,,
,Material subtotal,,,=SUM(Material),
Assembly,SMD placement,,,,"rate x placement count"
Assembly,Through-hole / hand,,,,
Assembly,Test / inspection,,,,
Assembly,Machine setup,,,,"per lot"
,Assembly subtotal,,,=SUM(Assembly),
Overhead,Approved basis and rate,,,,"record cost-model owner and source"
NRE,Stencil / fixture / jig,,,,"one-time, separate line"
,Cost subtotal,,,,"reconcile approved cost basis"
Commercial,Approved pricing method,,,,"margin or markup; record authority"
,Proposed unit price,,,,"review with commercial owner"
```

> Decide customer-facing cost detail, NRE, MOQ, lead time and validity under the approved commercial process. The internal layout is not an authorized quotation.
{: .prompt-tip }

---

## 2 — 8D / 5-Why RCA form

Symptoms first (see [RCA for Software]({% post_url 2026-05-27-rca-for-software-symptoms-before-hypothesis %})).

```markdown
# RCA — <part / assembly / system> — <date>
D1 Team:
D2 Problem (symptoms ONLY, no guesses):
   What / where / when / how many / how detected
D3 Containment (stop the bleeding):
D4 Root cause — 5-Why:
     1. Why? →
     2. Why? →
     3. Why? →
     4. Why? →
     5. Why? →   (root cause)
D5 Corrective action (owner + date):
D6 Verification (how do we KNOW it's fixed? evidence, not a log line):
D7 Prevention (poka-yoke / process / checklist update):
D8 Closure + lessons learned:
```

---

## 3 — Stage-gate checklist

A gate is a *decision*, not a status update. Each item is pass/fail before the build advances. (Why each gate exists: [NPI Stage Gate]({% post_url 2026-05-27-npi-stage-gate %}).)

```markdown
# Stage Gate — <project> — Gate <PoC / EVT / DVT / PVT / MP>
[ ] BOM frozen at rev ___ ; long-lead & single-source flagged
[ ] DFM/DFA review done; findings closed or risk-accepted (owner: ___)
[ ] Costing locked; quote sent; PO received
[ ] Test/inspection plan defined (FAI / AOI / ICT / functional)
[ ] First-article build + report
[ ] Docs issued: assembly drawing, SOP, work instruction
[ ] Risks logged with owners
Gate decision:  GO / NO-GO / GO-WITH-CONDITIONS
Decided by: ___   Date: ___
```

> A stage gate with no NO-GO outcome isn't a gate — it's a checkbox. Be willing to stop.
{: .prompt-warning }

---

## 4 — SOP / work-instruction skeleton

13 principles behind a usable SOP: [SOP Revamp Playbook]({% post_url 2026-05-27-sop-revamp-playbook %}).

```markdown
# SOP-<id> — <title>  (rev __, <date>)
Purpose:
Scope (applies to / does not apply to):
Tools & materials:
Safety / ESD / handling:
Steps (numbered, one action each, with a pass criterion per step):
  1.
  2.
Verification & sign-off:
Revision history:
```

---

Take the templates, set the OH/margin rules and rates to match your shop, and keep the external quote a single clean number.

For the manufacturing handoff, use the [Engineering Release Package review and downloadable templates]({% post_url 2026-09-09-engineering-release-package %}) to reconcile file revisions, population instructions and the scoped release decision.

For a proposed component substitution or other baseline change, use the [Engineering Change Control assessment and worked example]({% post_url 2026-09-09-engineering-change-control %}) to record impacts, verification, material disposition and effectivity.

After a pilot, use the [Pilot Build Exit Review and Open Issue Log]({% post_url 2026-09-09-pilot-build-exit-review %}) to reconcile unit histories and record the evidence needed for the next build.
