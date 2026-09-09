---
title: "Pilot Build Exit Review: What Evidence Is Needed Before the Next Build?"
date: 2026-09-09 12:20:00 +0700
categories: [NPI, Process]
tags: [pilot-build, readiness, first-pass-yield, rework, manufacturing, verification]
description: "Reconcile a fictional 20-unit pilot, separate first-pass results from recovery, and record the evidence and actions needed for the next build."
---

> **TL;DR** — A pilot exit review connects the configuration actually built, unit-level results, unresolved problems and production preparation to a scoped next-build decision. A passing retest changes the unit's current result; its first-attempt history remains part of the review.

The [Release Package]({% post_url 2026-09-09-engineering-release-package %}) defines what to build. [Engineering Change Control]({% post_url 2026-09-09-engineering-change-control %}) controls changes to that baseline. A pilot exit review examines what happened when the build was executed and what evidence is still needed before proceeding.

This article proposes a working review for a small NPI team. The 20-unit example, its issue descriptions and configuration identifiers are entirely fictional. The arithmetic is derived from the downloadable unit records; no company KPI or real test result is represented.

## Start with the Decision Being Requested

Name the next activity: another engineering trial, a limited pilot, a production lot, or shipment. Each can require different evidence. Approval to run another controlled trial does not automatically authorize production or shipment.

Record the product, variant, current released package, build lot and review cut-off. Then identify the actual PCB/BOM, firmware, fixture and test-procedure revisions used. List any deviations or rework instructions and their authorization evidence.

When the recorded and actual configurations disagree, establish which units are affected before combining results. A mixed-revision pilot can still provide information, but its populations and limitations must be visible.

## Reconcile Units Before Calculating Rates

Use one identifier per physical unit and retain its attempt history. Set the process boundary explicitly: a functional-test checkpoint, a defined assembly route, or another agreed operation. Do not present a checkpoint result as the yield of the entire factory process.

At the review cut-off, assign every unit a mutually exclusive current category. Examples include first-pass complete, complete after recovery, failed/held, pending an initial result, and recorded scrap. Tailor the categories to the route, and account separately for any unit transferred outside the reviewed population.

Keep quantities planned, entered, completed and unresolved visible. If initial processing remains incomplete, report that status and treat rates as provisional under a stated calculation rule. Do not drop pending units just to improve the denominator.

ASQ defines first-pass yield around units meeting the process requirements without rework, rerun or retest. Preserve those exclusions when labeling the metric. [Source: ASQ Quality Glossary, First pass yield](https://asq.org/quality-resources/quality-glossary).

## A Fictional 20-Unit Checkpoint

For this exercise, all **20 distinct units entered the same pilot verification checkpoint and completed a first attempt**. No units were scrapped, transferred or awaiting an initial result. The JSON records are invented pass/fail events, not instrument logs or measured values.

| Current category | Unit IDs | Units | Attempt history |
|---|---|---:|---|
| Passed on first attempt | DEMO-001 to DEMO-015 | 15 | PASS; no rework or retest within this checkpoint |
| Passed after rework | DEMO-016, DEMO-017 | 2 | FAIL, rework, PASS |
| Passed on retest without rework | DEMO-018 | 1 | FAIL, no unit rework, PASS |
| Still failed / held | DEMO-019, DEMO-020 | 2 | FAIL; unresolved at the cut-off |
| **Reconciled population** | **20 unique identifiers** | **20** | **No duplicate units in the category totals** |

![Twenty fictional pilot units grouped by first-pass, rework recovery, retest-only recovery and unresolved failure, with first-pass yield of 75 percent and current passing proportion of 90 percent](/assets/img/2026-09-09/pilot-build-outcomes.svg)
_Generated from the supplied synthetic records. Each square represents one unit. Current test status is separate from release authorization._

The arithmetic is:

```text
Checkpoint first-pass yield = 15 / 20 = 75%
Current passing proportion = (15 + 2 + 1) / 20 = 90%
Units with rework recorded = 2 / 20 = 10%
Units still failed / held = 2 / 20 = 10%

Unit reconciliation: 15 + 2 + 1 + 2 = 20
Distinct units retested: 3 (including the two reworked units)
Total test attempts: 20 initial + 3 additional = 23
```

The denominator for these unit rates is **20, not 23 test attempts**. The reworked population overlaps the retested population, so those two counts must not be added as separate unit categories. Recovery improves current completion while first-pass history remains 15/20.

Neither 75% nor 90% is a recommended acceptance threshold. Twenty units is an exercise size, not a sampling prescription. A real pilot's size and criteria must follow the objectives, risks, product requirements and applicable agreements.

## Examine the Failures Behind the Numbers

For this example, the five initially failing units link to three synthetic issue groups. Unit recovery and issue closure are separate questions.

| Issue | Observation in the scenario | What remains unresolved | Proposed responsible function |
|---|---|---|---|
| DEMO-I01 | DEMO-016/017 fail initially and pass after recorded solder rework | No verified root cause or evidence that the problem will be prevented in the next build | Manufacturing / quality |
| DEMO-I02 | DEMO-018 fails and then passes without unit rework | Product intermittency, test contact and other test-system effects have not been distinguished | Test engineering / design |
| DEMO-I03 | DEMO-019/020 show a reset symptom and remain failed | Root cause, affected scope and effective corrective action are unknown | Design / test engineering |

These are invented observations for the exercise, not findings from a customer PCB. No inspection image or test log is implied. In an actual review, attach the relevant logs, images, measurements and configuration identity. A DFx finding needs an analysis image and source/location evidence sufficient to reproduce it.

For each issue, distinguish **observation**, **hypothesis**, **confirmed cause**, **containment**, **corrective action** and **verification of effectiveness**. A note such as "probably fixture contact" belongs in the hypothesis field until supported by evidence.

Record the owner who has accepted the action, the affected units or builds, the closure trigger and the evidence required. The proposed functions above are planning suggestions; they do not represent assigned people or completed actions.

## Check Whether the Next Build Can Be Executed Consistently

Passing units show what those units did under the recorded conditions. The review also needs evidence about executing the next build.

| Review area | Evidence to seek for the intended next activity |
|---|---|
| Actual build configuration | Unit or lot traceability to the package, fitted parts and applicable software |
| Assembly instructions | Usable instructions, controlled deviations and lessons from actual build problems |
| Test and fixture | Applicable limits, fixture identity, equipment status and investigation of inconsistent results |
| Materials | Correct parts and quantities for the planned scope, with old/new stock and WIP dispositions resolved |
| Operators and handoff | Required training or demonstration, clear responsibilities and acknowledged current instructions |
| Execution time and capacity | Recorded setup, processing, waiting and rework time where needed for planning; unknown values stay unknown |
| Open risks | Named blockers, authorized restrictions and verification needed for closure |

A test-limit document by itself does not demonstrate that the test detects every relevant failure. Likewise, a few recovered units do not establish a stable process, a production capacity or a reliability claim. State what the pilot actually covered and what it did not.

## Make a Scoped Exit Decision

Use the organization's existing decision authority. The review coordinator can reconcile the records and collect specialist conclusions without creating an additional committee or acquiring other functions' approval powers.

| Working decision | Required meaning in the record |
|---|---|
| PENDING | Required review or authorization is incomplete |
| HOLD | The named next activity is blocked by unresolved requirements or evidence |
| GO WITH CONDITIONS | A permitted, precisely bounded activity has an authorized scope, restrictions, owners and closure triggers |
| GO | The evidence and required authorization support the named next activity |

For the fictional example, the **assessment recommendation is HOLD unrestricted production progression**: the reset failures remain unresolved and the no-rework retest has no established explanation. The current passing proportion does not resolve those questions. A separate controlled investigation or trial could be proposed, with its own objectives and authorization.

The formal decision remains **PENDING** in the example. No production, material disposition or shipment approval is invented. Closure must be based on the applicable criteria and evidence, rather than a majority vote of passing units.

When corrective work changes a part, firmware, fixture, limit or instruction, route it through [Change Control]({% post_url 2026-09-09-engineering-change-control %}) and update the relevant [Release Package]({% post_url 2026-09-09-engineering-release-package %}). Carry unresolved actions into the next review with their history intact.

## Keep the Review Useful for a Small Team

Maintain one review record and one open-issue log, linking to existing evidence rather than copying it into multiple documents. Before the meeting, reconcile the population and identify the decisions actually needed. At the meeting, resolve the scope, owners and next actions. Afterward, record the decision and verify the agreed closures.

To evaluate whether this method helps, try it on one authorized real project. Record preparation time, missing evidence discovered and repeated issues that were still open at the next build. Compare with an available previous approach while noting differences in product complexity. These are suggested evaluation measures, not claimed improvements or promised savings.

## Download the Working Files

- [Pilot Build Exit Review — editable Markdown]({{ "/assets/downloads/pilot-build-exit-review/pilot-build-review.md" | relative_url }})
- [Open Issue Log — editable Markdown]({{ "/assets/downloads/pilot-build-exit-review/open-issue-log.md" | relative_url }})
- [Synthetic 20-unit lot — JSON source data]({{ "/assets/downloads/pilot-build-exit-review/demo-lot.json" | relative_url }})
- [Template guide, metric rules and data fields]({{ "/assets/downloads/pilot-build-exit-review/template-guide.md" | relative_url }})

The Markdown records are blank. The JSON is populated demonstration data and must not be imported as actual production history. Use real requirements, evidence and authority when adapting the records to a build.
