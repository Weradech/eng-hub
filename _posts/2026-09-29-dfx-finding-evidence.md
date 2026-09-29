---
title: "A DFx Finding That Someone Else Can Verify"
date: 2026-09-29 12:00:00 +0700
categories: [NPI, DFM]
tags: [dfx, dfm, evidence, gerber, pcba, disposition]
description: "The minimum source, image, analysis and disposition needed for a reproducible PCB or PCBA DFx finding."
---

> **TL;DR** — A DFx finding needs an image from the actual analysis, an exact source and location, a measurable or observable condition, the affected operation and a requested decision. A checklist mark or generic illustration is not evidence.

This is a proposed evidence format. It does not set fabrication limits or approve a board. Compare against the **actual** customer requirement, stack-up, supplier process and agreed manufacturing method. Use the [DFM checklist]({% post_url 2026-06-29-dfm-checklist-pcba %}) to decide what to inspect; record each finding in a [blank issue sheet]({{ "/assets/downloads/dfx-finding/issue-sheet.md" | relative_url }}).

## Capture before recommending a fix

For every finding, save the inspected source file and revision or hash, viewer/tool version, relevant layer or drawing, coordinates or RefDes, and an image from the inspection. Mark the location and dimension on the image when possible. Keep the unmarked source view or file available so a second reviewer can reproduce the observation.

Write the observation separately from the inference:

| Field | Example of the question it answers |
|---|---|
| Observed condition | What is visible or measured, and where? |
| Requirement or process basis | Which controlled rule or supplier capability applies to this build? |
| Consequence | Which fabrication, assembly, inspection or test step could be affected? |
| Confidence and gap | What is certain from the file, and what still needs supplier or design-owner confirmation? |
| Proposed action | Who can change the design, confirm capability or accept a documented deviation? |

If the applicable limit is unknown, record **CAPABILITY UNCONFIRMED**. Do not turn a preferred target from another shop into a pass/fail rule. If the image is missing, mark **EVIDENCE INCOMPLETE**; do not publish the row as a supported finding.

## Close the loop on the exact revision

Assign an owner and a closure trigger. After a correction, inspect the new revision at the same location and update the source identity, image and result. If the proposal changes the released build, route it through [engineering change control]({% post_url 2026-09-09-engineering-change-control %}) and update the [release package]({% post_url 2026-09-09-engineering-release-package %}). Preserve the original finding and its evidence so the decision remains traceable.

The issue sheet separates **observation**, **analysis**, **recommendation**, **design response**, **verification** and **authorization**. A resolved image does not by itself authorize fabrication or production.
