---
title: "NPI Stage Gates: Record the Decision for the Next Activity"
date: 2026-05-27 13:00:00 +0700
categories: [NPI, Process]
tags: [npi, stage-gate, evt, dvt, pvt, product-development]
description: "A small-team stage-gate method that separates observed evidence, unresolved risks and authorized next-build scope."
---

> **TL;DR** — A gate records whether evidence supports a named next activity, for a named product configuration and scope. The criteria, quantity, schedule and authority come from the actual project. A passed check, a review recommendation and production authorization are different records.

![Illustrative NPI stages with machine result, engineering assessment, recommendation and authorized decision kept separate](/assets/img/2026-05-27/npi-stage-gate.svg)
_This is a proposed decision pattern, not evidence that a project passed a gate._

PoC, EVT, DVT, PVT and MP are useful stage names, but a high-mix, low-volume project may combine or rename activities. This article proposes a decision pattern, not fixed durations, yields, cost multipliers or mandatory sample sizes. Follow the customer's agreement and the organization's controlled process where they apply.

## Ask one scoped question at each handoff

| Example handoff | Decision question | Evidence to inspect |
|---|---|---|
| Concept to engineered prototype | Is the core concept demonstrated well enough to fund the next design step? | Demonstration conditions, unresolved assumptions, requirements and project risks. |
| Prototype to design validation | Does the tested configuration address the applicable design requirements? | Controlled design identity, requirement-linked analysis and test, failures and open changes. |
| Design validation to pilot | Can the intended supplier and process execute the defined build? | [DFM review]({% post_url 2026-06-29-dfm-checklist-pcba %}), confirmed capability, controlled instructions, test method and [release package]({% post_url 2026-09-09-engineering-release-package %}). |
| Pilot to next build or shipment | What did the actual units and process demonstrate, and what remains open? | [Pilot unit histories]({% post_url 2026-09-09-pilot-build-exit-review %}), deviations, failures, owner actions and authorized restrictions. |

The project's applicable reliability, regulatory, capacity and customer checks must be named in its gate plan. Do not assume every product needs the same tests. Do not fill a missing acceptance threshold with a generic percentage from a blog post.

## Keep the gate record small

Record the product and variant, current configuration, decision cut-off, requested next activity, evidence links, unresolved items, allowed restrictions and decision authority. A single coordinator can maintain the index and chase inputs. Design, quality, manufacturing, commercial and customer decisions remain with their existing owners.

Use four separate states:

1. **Machine result:** a tool checked a defined nonzero scope and reported its result.
2. **Engineering assessment:** a reviewer interpreted the result against the design requirements and open risks.
3. **Gate recommendation:** the team proposes HOLD, GO WITH CONDITIONS or GO for a named next activity.
4. **Authorized decision:** the person with the actual authority records the decision, conditions and date.

An empty check, an unresolved requirement, an unverified supplier capability or an unrecorded decision remains open. A conditional GO must name exactly what can proceed, for which quantity or lot, under which restriction and with what closure trigger. It cannot silently authorize production or shipment.

## Reopen when the baseline changes

If the BOM, design, firmware, fixture, method or supplier changes, assess which evidence and decision still apply. Link the change through [engineering change control]({% post_url 2026-09-09-engineering-change-control %}) and preserve the earlier gate record. The [fictional RFQ-to-pilot thread]({% post_url 2026-09-29-one-build-from-rfq-to-pilot %}) demonstrates where a gate stops when the needed record is absent.
