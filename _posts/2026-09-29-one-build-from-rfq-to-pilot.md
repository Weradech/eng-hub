---
title: "One Build, Four Decisions: A Fictional RFQ-to-Pilot Thread"
date: 2026-09-29 11:00:00 +0700
categories: [NPI, Process]
tags: [rfq, release, pilot-build, stage-gate, configuration-management]
description: "A single fictional product thread showing the records and stop points from quote intake to the next-build decision."
---

> **TL;DR** — One product identifier must survive RFQ, release, pilot and review. Each stage asks a different question and leaves a record for the next one. Missing evidence stops the affected decision; a later stage cannot retroactively approve an earlier one.

This entire thread is a **synthetic teaching exercise**. `DEMO-CTRL`, variant `BASE`, package `DEMO-PKG-A` and pilot `DEMO-PILOT-001` are fictional identifiers. There is no customer order, controlled design, measured production result or authorized release behind them. The linked articles contain independent examples; their records must not be mistaken for one approved history.

| Stage | Question | Working record | Stop point |
|---|---|---|---|
| RFQ | What exactly is being priced? | File and assumption register | A missing variant or test scope blocks a firm commitment for that scope. |
| Release review | Do the exact files describe one build? | Checklist and manifest | Conflicting BOM, placement and drawing instructions hold assembly release. |
| Pilot execution | What happened to the identified units? | Configuration and unit attempt histories | Incomplete or mixed histories limit what rates can mean. |
| Exit review | What activity may happen next? | Issue log and scoped decision record | Unresolved failures or missing authority leave that decision pending. |

## 1. Intake: define the quote boundary

The fictional request names `DEMO-CTRL/BASE` but initially leaves population and test scope to be confirmed. The intake record lists the actual customer files, quantity basis, assembly operations, programming and test assumptions. The estimator can prepare conditional scenarios, but a firm price or delivery promise for the unresolved work waits for the responsible answer. See the [RFQ intake method]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}). No price is invented for this exercise.

## 2. Release: reconcile the intended configuration

The [release article]({% post_url 2026-09-09-engineering-release-package %}) gives a fictional conflict: BOM says R12 is DNP, placement output includes R12, and assembly drawing says fit R12. Its justified conclusion is **HOLD assembly release** until the design owner resolves the intended population and affected files are corrected and rechecked. This article does not change that recorded conclusion.

For the later training stage, **assume a separate, hypothetical branch** in which the conflict is resolved, the applicable checks are repeated and an authorized person issues `DEMO-PKG-A`. No such approval record or corrected file set is supplied. Therefore that branch illustrates the handoff only; it cannot be used as a release example or evidence that the release gate passed.

## 3. Pilot: reconcile physical units

The [synthetic pilot dataset]({{ "/assets/downloads/pilot-build-exit-review/demo-lot.json" | relative_url }}) names `DEMO-CTRL/BASE`, `DEMO-PKG-A` and 20 fictional unit IDs. At one defined checkpoint, 15 passed first attempt, two passed after rework, one passed on retest without rework and two remained failed. These four exclusive groups total 20. Checkpoint first-pass yield is 15/20; the current passing count is 18/20. Neither number is a real company KPI, a whole-factory yield or an acceptance target. See the [pilot review method]({% post_url 2026-09-09-pilot-build-exit-review %}) for the arithmetic and its limits.

The dataset identifies a package, firmware, test procedure and fixture only by fictional labels. It contains no actual test logs, drawings, source files or signed release. The training branch therefore cannot establish that its configuration was technically correct or authorized.

## 4. Next decision: name the permitted activity

The two still-failed units and the retest-only recovery require investigation. Unit recovery does not close an underlying issue. Record each observation, hypothesis, affected population, owner, required evidence and closure trigger in the [open-issue log]({{ "/assets/downloads/pilot-build-exit-review/open-issue-log.md" | relative_url }}). Link applicable requirements to the evidence through a [verification matrix]({% post_url 2026-09-29-requirements-to-verification-traceability %}).

The training dataset leaves `formal_decision`, approved next scope and decision authority pending. The defensible statement is **the next-build authorization is not recorded**. An authorized decision could later allow another controlled trial or another defined activity under its own conditions; this page grants none.

## Reuse the thread on a real project

Copy only the needed records from the [template index]({{ "/templates/" | relative_url }}). Replace every fictional identifier with the controlled product and variant. Link actual source files and analysis images. Reconcile quantities and file revisions, keep unknowns visible, and request decisions from the existing authority. A complete set of filled files is still not a manufacturing approval unless the evidence and decision support the stated scope.
