---
title: "A Traceability Gap on the Line: Which Units Should We Hold?"
date: 2026-11-22 12:00:00 +0700
categories: [Manufacturing, Quality]
tags: [traceability, hold, containment, mes, rework, evidence]
description: "How to set, narrow and close a hold when a unit's traceability record is missing or contradicts the physical product."
---

> **TL;DR** — When a traceability record is missing or wrong, first describe the gap without guessing its cause. Hold every unit you cannot yet exclude, using a boundary anyone can apply. Write the release criteria before reviewing evidence. Release units one by one against those criteria, widen the scope when evidence points outside it, and close only when every unit in scope has an approved, traceable outcome.

This is a proposed method, not a company procedure. It does not replace the organization's controlled hold, nonconformance or customer-notification rules. The case below is **fictional**. Record each hold in a [blank hold scope record]({{ "/assets/downloads/traceability-hold/hold-scope-record.md" | relative_url }}).

## When to use it

Use it when the recorded history of a unit cannot be trusted for a decision. For example:

- a board has no active binding to the finished unit it is physically inside;
- two finished units claim the same board;
- a required station result is missing;
- an operator says a scan "did not go in", and the system shows nothing;
- a label exists for a serial that the system never issued;
- station records were still pending when units moved on after a network outage.

The [DMC traceability pattern]({% post_url 2026-08-26-shopfloor-dmc-family-traceability %}) describes how these records are created. This article covers what to do when one of them is wrong.

## 1. Describe the gap before explaining it

Write what was observed, where and when, and how you know. Keep cause out of this step.

| Observation | Not yet an observation |
|---|---|
| "At final assembly, the MES shows no active main-board binding for unit DEMO-FU-0107." | "The rework bench forgot to scan." |
| "The operator reports scanning the board twice; the station shows no record." | "The scan was never made." |

"Not found in MES" has more than one meaning. The event may not have happened. Or the station may have refused the scan before anything was sent, and some station screens refuse input without logging it. Until the station's own logs are checked, record the question as **UNKNOWN**.

## 2. Set a boundary anyone can apply

The initial scope is every unit that could share the cause and that you **cannot yet exclude**. Define it with a recorded attribute, not a list of names that someone remembers:

- same station and time window;
- same rework batch or bench;
- same work order or label roll;
- the period when station records were pending.

Then search beyond the line. Units already packed, in transit or shipped are part of the scope. List them separately, because acting on them needs different authority. Whether a customer must be informed is a decision for the named authority in the organization's procedure. The engineer's job is to provide the list and the evidence.

For every unit left outside the boundary, record the evidence that excludes it. Review the boundary whenever new evidence arrives. A boundary that is too narrow can miss a unit that is hard to find later.

## 3. Write the release criteria first

Before evaluating any evidence for release, write what a unit needs in order to be released, and record who wrote the criteria and when. For a binding gap the criteria might be:

1. **Binding:** the board DMC read physically inside the unit matches its active binding. If it does not, the binding is corrected by closing the wrong row with a reason and adding a new one. The old row is not overwritten.
2. **Results:** every test result the route requires for the board is recorded as PASS for the configuration in use.
3. **Failures:** any recorded FAIL for the board has a conclusion accepted by an authorized person, or an approved disposition. An analysis that is still open does not count.
4. **Authority:** an authorized reviewer releases the unit. Correcting the genealogy is not a release.

Writing the criteria first reduces the risk that the evidence reshapes the rule. A unit that cannot meet them stays held and goes to disposition.

## 4. Use evidence for what it can show

Each source answers a limited question.

| Evidence | What it can establish | What it cannot establish |
|---|---|---|
| Physical read of the board DMC inside the unit | What is in the unit now | What was in it before a later rework |
| Binding history | What the system recorded and when | That the recorded binding matches the hardware |
| FCT pass for a board | The board passed the configured test | Which finished unit the board is inside |
| Rework log | Which units were opened, by whom, when | That every swap was recorded |
| Stores issue log | Which serials were issued, to which location, when | That no board reached the line without an issue record, or which unit received it |
| Station or client logs of refused scans | That a scan reached the station and why it was refused | Nothing, if refusals are not logged; the question stays UNKNOWN |
| Label print log | Which labels were printed, and when | That each label went onto the intended unit |
| Pending queue after an outage | Which records had not yet been stored or rejected | Which units moved on while they were pending |

Record every source in the evidence log, including the ones that did not help. An empty result is evidence only if the source would have recorded the event.

## 5. Let the scope change, and record why

Evidence can remove units from the scope, but it can also point outside it. When a board turns up somewhere the boundary did not cover, ask what else that new place could have affected, and widen the boundary to it. Every scope change needs a reason, the evidence, and the person who accepted it.

## A fictional case

During one afternoon, rework bench A replaced main boards in **12** units of a product. At final assembly, unit DEMO-FU-0107 showed no active main-board binding.

**Gap.** The binding history showed that the old board was unbound, but no new binding was recorded. The bench operator reported scanning the new board. The bench client did not log refused scans. Whether the scan reached the station: **UNKNOWN**.

**Initial scope.** All 12 units reworked at bench A in that time window. Packing records showed that 3 of them were in a packed carton still on site. None had shipped. Total: 12 units, all held.

**Release criteria.** The four criteria in step 3, written and dated before any unit was evaluated for release.

**First evidence.** The team read the board DMC inside all 12 units, opening the carton under the hold.

- In 11 units, the board matched the binding recorded at bench A.
- In DEMO-FU-0107, the board was DEMO-MB-0418. The system showed DEMO-MB-0418 bound to DEMO-FU-0112, a unit reworked the same day at bench B.

**Scope change.** The evidence showed boards crossing between benches, so bench B's rework that day could share the cause. Bench B's rework log listed 4 units, including DEMO-FU-0112, all still on the line. All 4 were added: scope 16.

**Excluding other units.** The MES could not be the only basis, because its rework records were the thing in doubt. The team used an independent source: the stores issue log, which records every main-board issue to any location, listed 16 replacement main boards issued that day, by serial, all of them to benches A and B. The 16 boards removed during rework were counted in the benches' quarantine bin, by serial. A binding-history query found no other rework unbinds that day; it was used only as a cross-check. The search would stay open if any issued board were not found in a unit in scope.

**Second evidence.** Physical reads of bench B's 4 units: 3 matched their bindings. DEMO-FU-0112 contained board DEMO-MB-0420, which had no binding at all. All 16 boards on the issue log were found in the 16 units in scope, so no other unit could have received one of them. The two units had probably had their boards swapped during rework. The records alone could not confirm that.

**Record correction (not a release).** The binding of DEMO-MB-0418 to DEMO-FU-0112 was closed with the reason "physical read: board is inside DEMO-FU-0107". New bindings were added for DEMO-MB-0418 to DEMO-FU-0107 and DEMO-MB-0420 to DEMO-FU-0112, each citing the physical read.

**Decision (fictional).**

- The 14 matching units met all four criteria and were released by the authorized reviewer.
- DEMO-FU-0112 met criterion 1 after correction. DEMO-MB-0420's required results were recorded as PASS, with no FAIL recorded. The authorized reviewer released it.
- DEMO-FU-0107 met criterion 1 after correction, but failed criterion 3. DEMO-MB-0418 had a first-attempt FCT FAIL followed by a PASS on retest, with no accepted conclusion on the FAIL and no approved disposition. It stayed held, and a disposition was requested.

**Status.** 16 units in scope = 15 released + 0 scrapped + 0 other approved disposition + 1 still held. The hold stays **OPEN** until DEMO-FU-0107 has an approved, traceable disposition.

**Prevention.** A change was requested under engineering change control for the bench client to record every refused scan, with the value, station, time and reason, where an investigation can read it.

## 6. Close on a count

Give every unit in scope exactly one current outcome. Rework is part of a unit's history, not an outcome: a reworked unit that is then released counts once, as released.

```text
units in final scope = released + scrapped + other approved disposition + still held
```

Close the hold only when **still held** is zero and each outcome is approved and traceable under the organization's procedure. Then record the cause (confirmed or suspected, with evidence) and the prevention action, with an owner and a way to verify it. Use [engineering change control]({% post_url 2026-09-09-engineering-change-control %}) if the prevention action changes a released instruction or station software.

## Evidence rule

Absence of a record is not evidence that nothing happened. A passed test is not proof of which unit the board is in. A corrected record is not a release. Leave the decision fields **PENDING** until an authorized person makes the decision on the evidence attached.
