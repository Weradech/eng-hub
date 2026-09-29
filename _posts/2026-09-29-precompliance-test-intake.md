---
title: "Pre-compliance Lab Intake: What Can This Test Actually Claim?"
date: 2026-09-29 13:00:00 +0700
categories: [NPI, Test]
tags: [pre-compliance, laboratory, test-evidence, measurement, configuration]
description: "A small-lab intake and evidence record that keeps exploratory findings separate from formal certification or accreditation claims."
---

> **TL;DR** — Agree on the equipment under test, configuration, question, method, conditions and report scope before running a pre-compliance test. Preserve setup and raw results. State plainly what was observed, what remains untested and whether the result is only exploratory.

This is a proposed working method for a small laboratory service. It does **not** claim the lab is accredited, recognized by a regulator or authorized to certify a product. A formal compliance route depends on the particular market, product and scheme; confirm that route with the responsible authority or certification body before promising a formal deliverable.

## Intake: define the question and the boundary

Record the customer request, intended market or use, applicable requirement source and revision, and the decision the customer hopes to make. Identify the equipment under test (EUT): model, serial, PCB/BOM, firmware, fitted options, accessories, power supply, cables, load and operating mode. If the customer's request is only a diagnostic scan, label it as such. Do not call an unspecified scan a full compliance test.

| Intake question | Record before testing |
|---|---|
| What is being tested? | Exact EUT and configuration, including any customer-supplied changes. |
| Against what? | Controlled requirement, limit or agreed exploratory objective; source and revision. |
| Under which conditions? | Setup, environment, operating mode, input, load and test boundaries. |
| By which method? | Procedure revision, instrument, fixture, software and responsible operator. |
| What is excluded? | Untested modes, standards, configurations, frequencies or conditions. |
| What may be reported? | Raw observations, engineering assessment and any permitted conformity statement, kept separate. |

If the limit, method, configuration or service capability is unknown, clarify it before accepting a pass/fail deliverable. A customer-provided limit remains attributed to its source; the lab does not invent one.

## Execution: keep the result reproducible

Capture setup photos, connection diagram, instrument and software identities, calibration status where relevant, environmental conditions, raw files, settings, date/time and any deviation. Keep preliminary scans distinct from a final run. Identify retests and changes to the EUT. A later firmware change can make an earlier result inapplicable to the new configuration.

Traceability is a property of a **measurement result**, not a sticker on an instrument or a general property of a laboratory. The [NIST traceability guidance](https://www.nist.gov/metrology/metrological-traceability) explains the documented chain and supporting measurement information needed for a traceability claim. Record uncertainty and decision rules when they matter to the stated comparison; do not infer them from a calibration due date alone.

## Report: separate observation, assessment and authorization

Use a compact report with the request ID, EUT configuration, method, conditions, exact result references, deviations and limitations. State whether a number is a measured result, a calculated value or an operator observation. If a criterion was agreed, record the comparison and its decision rule. An exploratory test may end with a risk statement or next-test recommendation, not a certification claim.

ISO describes [ISO/IEC 17025](https://www.iso.org/standard/66912.html) as requirements for the competence of testing and calibration laboratories. Referring to that standard here does not claim this lab conforms to it or has accreditation. Use the customer's applicable scheme and existing quality system to decide what formal report controls are required.

Copy the [blank test intake and report record]({{ "/assets/downloads/precompliance-lab/test-intake-record.md" | relative_url }}). It leaves result and approval fields empty. If testing reveals a PCB/PCBA issue, create a [DFx finding with an actual analysis image]({% post_url 2026-09-29-dfx-finding-evidence %}) and route design changes through the controlled baseline.
