---
title: "Requirements to Verification: Can the Planned Test Answer the Question?"
date: 2026-09-29 10:00:00 +0700
categories: [NPI, Test]
tags: [requirements, verification, fct, ict, traceability, pilot-build]
description: "A small requirements-to-evidence matrix for planning checks and keeping method, result and approval separate."
---

> **TL;DR** — Give each applicable requirement an identity, condition, limit and verification method. Link the actual result to the exact product configuration. A planned test is not a passed test, and a passed sample does not automatically cover an untested operating range.

This is a practical proposal for high-mix, low-volume NPI. Use the customer specification and the organization's approval rules as the authority for the real product. The fictional row below is a teaching example, not a test result.

Copy the [blank verification matrix]({{ "/assets/downloads/verification/requirements-verification-matrix.md" | relative_url }}) when a controlled requirement set is available. Leave its result and decision fields pending until supported.

## Start with the requirement, not the equipment list

Copy the exact controlled requirement reference and revision into a compact matrix. Preserve the wording in its source; the matrix can summarize but must link back. Record applicable variant, operating conditions, measurable limit and who owns interpretation when the requirement is ambiguous.

| Field | What it answers |
|---|---|
| Requirement ID and source revision | Which obligation is being checked? |
| Product, variant and condition | Where does it apply? |
| Acceptance criterion | What result would satisfy it? |
| Method and coverage | Analysis, inspection, ICT, FCT or another check; which part of the requirement can it observe? |
| Test configuration | Board/BOM, firmware, fixture, instrument and procedure revisions. |
| Evidence reference | Raw result, calculation, image or report location. |
| Status and reviewer | NOT RUN, PASS, FAIL, BLOCKED or N/A with reason; who reviewed it? |

For example, a **fictional** requirement `DEMO-REQ-01` might ask for a sensor output within a stated range at specified input and temperature conditions. A room-temperature reading at one input is evidence for that condition only. The remaining range needs a defined analysis or test plan. No result is supplied here.

## Match the method to the failure it can detect

An ICT point may confirm accessible electrical connectivity or a measurable component property. It does not by itself demonstrate firmware behavior or full product function. An FCT step can exercise a specified function under its configured conditions; it cannot prove every untested condition. Visual inspection and AOI answer different questions again. Use the [ICT assessment]({% post_url 2026-05-27-ict-assessment-pattern %}) and [FCT method]({% post_url 2026-05-29-fct-methodology-deep-dive %}) to select candidate checks, then document each coverage gap.

For every method, state the stimulus, measurement, equipment state, limit and failure response before running it. If a fixture or software version changes, assess whether the old result still applies. Retain first attempts and retests separately; the [pilot exit review]({% post_url 2026-09-09-pilot-build-exit-review %}) shows why a recovered unit cannot be counted as a first-pass unit.

## Keep four states separate

1. **Requirement identified:** source and applicability are known.
2. **Method planned:** the check and criterion are defined, with gaps visible.
3. **Evidence recorded:** actual configured execution or calculation exists.
4. **Decision made:** an authorized person accepts the evidence for a named activity.

If a requirement changes, trace it through the verification plan, released package, tested units and open issues. Use [engineering change control]({% post_url 2026-09-09-engineering-change-control %}) for the affected baseline. Leave the decision pending when evidence or authority is missing.
