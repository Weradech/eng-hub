---
title: "Engineering Change Control: What Changes When One Component Changes?"
date: 2026-09-09 11:53:00 +0700
categories: [NPI, Process]
tags: [engineering-change, bom, configuration-management, pcba, verification, traceability]
description: "Follow a component change from request to implementation, with a calculated resistor-tolerance example and editable change assessment records."
---

> **TL;DR** — A component change needs an impact assessment, evidence against the affected requirements, an authorized decision and a defined implementation boundary. Record which configuration is allowed on which builds, and verify that the factory actually uses it.

![Engineering change control from a defined request through impact review, verification, decision, implementation and closure](/assets/img/2026-09-09/engineering-change-control.svg)
_A proposed workflow for a small NPI team. The example and all component identifiers in this article are fictional._

The [Engineering Release Package]({% post_url 2026-09-09-engineering-release-package %}) article explains how to issue a consistent build configuration. This article addresses what happens when that configuration needs to change.

For terminology, this workflow uses **Engineering Change Request (ECR)** for the proposed change and **Engineering Change Order (ECO)** for the authorized implementation instruction. Organizations use these terms differently. A small team can hold both in one record with distinct request, decision and implementation sections, using its existing approval rules.

NASA's configuration-management guidance describes identifying a baseline, controlling changes and maintaining records of configuration status and verification. Those are useful organizing principles; the lightweight records below are an editorial proposal for manufacturing work, not a NASA process requirement imposed on an EMS business. [Source: NASA, Configuration Management](https://www.nasa.gov/reference/6-5-configuration-management/).

## 1. Define the Change Against a Known Baseline

"Use the available equivalent" leaves too much unspecified. Start with the product, variant, reference designator, current approved part and candidate part. Link the currently released package and state why the change is requested.

| Request field | What it should resolve |
|---|---|
| Current configuration | Released package ID, BOM revision, variant and affected reference(s) |
| Proposed configuration | Candidate manufacturer/part number and actual changed characteristics |
| Reason | Shortage, obsolescence, correction, performance or cost opportunity, with supporting evidence |
| Requested timing | Proposed order, lot or serial boundary; keep it separate from approved effectivity |
| Scope | Evaluation only, a limited deviation, or a continuing baseline change |

Preserve the released baseline while the candidate is under evaluation. Sample purchasing or a controlled engineering trial can have its own authorization; neither action by itself authorizes substitution on production orders.

## 2. Review the Interfaces Around the Component

Assign only the reviewers needed for the affected interfaces. The NPI coordinator collects evidence and tracks actions; the existing design, manufacturing, quality, commercial or customer authorities make decisions within their responsibilities.

| Area | Questions to answer | Evidence to link |
|---|---|---|
| Circuit and PCB | Do electrical limits, tolerance, pinout, package details and operating conditions remain acceptable? | Controlled datasheet comparison, calculations and relevant design checks |
| Firmware and calibration | Do scaling, thresholds, timing, drivers or stored calibration depend on the changed characteristic? | Impact review and applicable regression/calibration results |
| Assembly and inspection | Do placement, polarity, soldering, handling or inspection instructions need revision? | Manufacturing review and applicable trial evidence |
| Test and fixture | Can the current method detect the affected failure? Are limits and fixture compatibility still valid? | Requirement-to-test mapping, fixture identity and result records |
| Material and work in progress | Where are the original and candidate parts, and which builds already contain them? | Stock, kit, WIP and finished-goods records |
| Customer and commercial commitments | Does the change affect an approved BOM, specification, delivery promise or contractual approval requirement? | Applicable agreement, customer decision or documented N/A reason |
| Product compliance | Does the change affect an applicable safety, environmental or certification condition? | Review against the actual product requirements and supporting evidence |

Use **NOT REVIEWED**, **AFFECTED**, **UNAFFECTED WITH EVIDENCE** or **N/A WITH REASON**. An empty cell represents unfinished work. Matching resistance and package size only answer part of the first row.

## 3. Worked Example: A 10 kΩ Resistor with a Wider Tolerance

**Fictional change request DEMO-ECR-001:** replace the upper resistor R1 in a sensing divider because an alternative is available. No real manufacturer part, customer board or measured result is represented.

The example deliberately isolates initial resistor tolerance:

- Input is an ideal, fixed **3.300 V**.
- R1 and R2 are both nominally **10 kΩ**. R1 is the upper resistor; R2 connects the output node to ground.
- Original R1: **±1%**. Candidate R1: **±5%**. R2 stays at **±1%**.
- Both R1 options are described as 0603 for the example. Other characteristics remain unverified.
- The **invented exercise requirement** is **1.650 V ±2%**, or **1.6170–1.6830 V**, for this resistor-tolerance screen.
- Loading, input-voltage variation, temperature drift, aging and measurement/ADC errors are excluded. This is an initial calculation, not a complete circuit qualification.

For an unloaded divider:

```text
Vout = Vin × R2 / (R1 + R2)
Vout_min = Vin × R2_min / (R1_max + R2_min)
Vout_max = Vin × R2_max / (R1_min + R2_max)
```

TI discusses how resistor ratio and input leakage affect divider accuracy. Its application report supports the need to consider loading separately; it does not supply this article's fictional requirement or results. [Source: TI, SLVA450B, sections 2 and 4](https://www.ti.com/lit/an/slva450b/slva450b.pdf).

| Configuration | R1 range | R2 range | Calculated Vout range | Initial screen |
|---|---|---|---|---|
| Original: both ±1% | 9.9–10.1 kΩ | 9.9–10.1 kΩ | 1.6335–1.6665 V | Within the exercise limits for modeled tolerance only |
| Candidate: R1 ±5% | 9.5–10.5 kΩ | 9.9–10.1 kΩ | 1.6015–1.7005 V | Extends outside the exercise limits |

![Calculated tolerance intervals for the fictional divider, showing the candidate interval extending beyond the exercise limits](/assets/img/2026-09-09/change-tolerance-analysis.svg)
_Calculated evidence from the stated inputs, rounded to four decimal places. The shaded band is the exercise requirement; this is not measured production data._

The nominal voltage is 1.650 V in both cases. For the candidate, the extreme values are approximately **−2.94% and +3.06%** relative to nominal. A test on one nominal sample could miss that spread.

**Assessment recommendation: HOLD this candidate for production substitution.** Its tolerance range does not guarantee the exercise requirement under the simplified model. This does not mean every candidate resistor would fail, and it is not a recorded company approval decision. The downloadable worked example leaves formal decision fields pending.

Possible next actions are to evaluate a suitable candidate with tighter tolerance, redesign the sensing approach, or propose a justified requirement change to its authorized owner. Each path needs its own evidence. Widening the FCT acceptance window merely to make the candidate pass would leave the original requirement unresolved.

## 4. Define Verification Before Running a Trial

For each affected requirement, record the method, operating conditions, sample selection rationale, acceptance criteria and reviewer before testing. Use analysis where it can establish a bound and testing where actual behavior must be demonstrated.

For the divider example, a complete evaluation would also consider the real input range, input loading, temperature behavior and the downstream measurement or threshold function. Any firmware compensation proposal needs its own assumptions, limits and verification.

Record the tested configuration: part identity, PCB/BOM revision, firmware, fixture, equipment and procedure. Keep calculated bounds and measured results in separate fields. A small pilot can check assembly execution; its sample size does not automatically establish all electrical or reliability limits.

Link screenshots, plots and source files to findings. A DFx finding requires the actual analysis image and enough file/location information to reproduce the observation. The plot above supports only the fictional tolerance calculation.

## 5. Decide What Happens to Existing Material

Changing the future BOM leaves existing material and assemblies to be addressed. Review each population separately.

| Population | Decision to resolve |
|---|---|
| Original parts in stock | Retain for an authorized baseline, reserve for another approved use, or seek a separate disposition decision |
| Candidate parts | Identify evaluation stock and prevent unintended production issue until authorized |
| Kits and open orders | Determine whether to continue, change or hold each affected order under the approved instruction |
| WIP and finished assemblies | Identify actual fitted parts; decide whether existing acceptance remains valid or whether additional work is required |
| Delivered units, if affected | Assess applicability with the responsible quality/customer authority and record any required follow-up |

Age, availability or the existence of a new part number does not decide scrap or rework. Record quantities, location, configuration, intended disposition and the person authorized to approve it. Where cost or demand is unknown, leave it unknown and assign an action rather than entering zero.

## 6. Set an Effectivity Boundary People Can Execute

**Effectivity** identifies the builds to which the approved change applies. Use a controlled order, lot or serial boundary that the factory can trace. A calendar date alone may leave old kits and WIP ambiguous.

An implementation instruction should identify:

1. The last permitted use of the previous configuration and first permitted use of the new one, where applicable.
2. Variant, quantity/lot restrictions and whether mixed configurations are permitted.
3. Updated documents, software and test records, plus unchanged items explicitly carried forward with justification.
4. Kit/WIP disposition, material identification and responsible implementers.
5. The first affected build to inspect and the evidence required to close implementation.

Keep **proposed effectivity** separate from **approved effectivity**. For the fictional HOLD example, approved effectivity remains empty and the candidate is not added to a released build instruction.

A temporary deviation also needs its permitted scope, expiry or closure trigger, traceability and a decision about what happens afterward. Returning to the previous configuration requires confirmation that it is still valid for the affected builds; it is not an automatic rollback instruction.

## 7. Release the New Configuration and Verify Its Use

After the required evidence and authorization exist, update the affected controlled records and issue a new [Release Package]({% post_url 2026-09-09-engineering-release-package %}). List changed files and carry-forward documents in its manifest. Preserve the previous package and the change record linking the two.

Check the first affected build against the new instruction: actual fitted part, material traceability, document revision, applicable software/test settings and order/lot identity. Record receipt of the changed instructions by the responsible functions. Close the change only when the implementation evidence and required follow-up are complete under the applicable process.

The example in this article does not reach this step. It supplies a failed calculation screen and the actions needed for further evaluation, without inventing a successful trial or authorization.

## Download the Working Records

- [Change Impact Assessment and implementation record — Markdown]({{ "/assets/downloads/engineering-change-control/change-impact-assessment.md" | relative_url }})
- [Structured change record — JSON]({{ "/assets/downloads/engineering-change-control/change-record.json" | relative_url }})
- [Worked example with inputs, calculations and open actions — Markdown]({{ "/assets/downloads/engineering-change-control/worked-example.md" | relative_url }})
- [Template instructions and JSON field definitions]({{ "/assets/downloads/engineering-change-control/template-guide.md" | relative_url }})

Start with the Markdown assessment. The JSON is optional; if both are used, identify the authoritative record and keep the representations consistent. Blank templates contain no approval or measured result. Adapt the review scope to the actual risk and existing authority rules rather than creating a second approval system.

After executing a controlled build, use the [Pilot Build Exit Review]({% post_url 2026-09-09-pilot-build-exit-review %}) to reconcile first-pass results, recovery and unresolved issues before deciding the next activity.
