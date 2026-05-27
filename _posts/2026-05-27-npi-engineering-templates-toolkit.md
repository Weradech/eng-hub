---
title: "NPI Engineering Templates — Copy-Paste Toolkit"
date: 2026-05-27 20:00:00 +0700
categories: [NPI, Process]
tags: [npi, templates, rca, stage-gate, sop, costing]
---

> **TL;DR** — The *why* behind these is already on this site. This post is the *what to paste into your sheet*: an RFQ costing layout, an 8D/5-Why RCA form, a stage-gate checklist, and an SOP skeleton. Adapt the rates and rules to your own shop.

---

This is a companion page, not a tutorial. Each template links back to the deep-dive that explains it:

- Costing → [PCBA RFQ Costing Pattern]({% post_url 2026-05-27-pcba-rfq-costing-pattern %})
- Stage gates → [NPI Stage Gate]({% post_url 2026-05-27-npi-stage-gate %})
- SOPs → [SOP Revamp Playbook]({% post_url 2026-05-27-sop-revamp-playbook %})
- RCA → [Buck LED Driver RCA]({% post_url 2026-05-27-buck-led-driver-rca %}) · [RCA for Software]({% post_url 2026-05-27-rca-for-software-symptoms-before-hypothesis %})

---

## 1 — RFQ costing sheet (internal working copy)

Copy into a spreadsheet. The customer sees only the bottom line — never the OH/Margin rows. (Rules + Thai EMS reference rates: see the [costing pattern post]({% post_url 2026-05-27-pcba-rfq-costing-pattern %}).)

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
Overhead,OH 15% (mfg cost only),,,=0.15*(Material+Assembly),"NOT on NRE"
NRE,Stencil / fixture / jig,,,,"one-time, separate line"
,Cost subtotal,,,=Material+Assembly+OH,
Margin,Margin (internal),,,,"NEVER shown to customer"
,UNIT PRICE (lump sum),,,=(Cost subtotal)/(1-Margin%),"only number that leaves the building"
```

> **Golden rule:** customer quotation = lump sum + NRE line + MOQ + lead time + validity. Expose nothing else.
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
