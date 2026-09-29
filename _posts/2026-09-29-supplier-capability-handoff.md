---
title: "Supplier Capability Handoff: Turn a DFM Question into a Recorded Answer"
date: 2026-09-29 14:00:00 +0700
categories: [NPI, DFM]
tags: [supplier, dfx, dfm, fabrication, assembly, release]
description: "A compact way to ask the selected manufacturer about an exact feature, preserve conditions and feed the answer into the release decision."
---

> **TL;DR** — Ask the selected manufacturer about the exact product revision, feature, quantity and process. Keep its answer, limits and conditions linked to the source image. A supplier saying “we can build it” does not approve the design or release the package.

This is a proposed handoff for a high-mix, low-volume team. Use the [DFM review]({% post_url 2026-06-29-dfm-checklist-pcba %}) to identify the process question and the [DFx evidence method]({% post_url 2026-09-29-dfx-finding-evidence %}) when there is a finding. No generic fabrication target on this site replaces the selected supplier's documented capability for this build.

## Ask one bounded question

Send product and variant, board or assembly revision, intended quantity, panel arrangement if relevant, process step, affected feature and actual design view. Include the source file identity, layer or RefDes, coordinates, measured condition and the proposed requirement or drawing note. Ask whether the supplier can meet **that condition under that process**, and request its documented limits, conditions, cost or lead-time impact and any needed design change.

Avoid a vague “is the board manufacturable?” answer. A supplier may accept a feature only with a particular stack-up, plating, stencil, fixture, inspection method or panel support. Record those conditions with the response date and named contact.

## Classify the answer before using it

| Response | Next action |
|---|---|
| Confirmed for the exact configuration and process | Attach the response to the check. Verify that drawings and release instructions carry the stated conditions. |
| Feasible with conditions or extra cost | Assess the design, commercial and schedule impact; obtain the required decision before release. |
| Not feasible | Raise a design change or choose a supplier/process that can meet the requirement. Keep the affected activity on HOLD. |
| No answer or ambiguous answer | Mark capability unconfirmed. Do not substitute a preferred rule or assume acceptance. |

The supplier owns its process-capability statement. The design owner owns the design response. The existing release authority decides whether the reviewed configuration may be issued. One NPI coordinator can track all three without inventing an extra approval layer.

## Close the handoff

If the response requires a change, update the source design and applicable outputs, then recheck the affected finding and supplier condition. Route a released-baseline change through [engineering change control]({% post_url 2026-09-09-engineering-change-control %}). Add the final response reference and conditions to the [engineering release package]({% post_url 2026-09-09-engineering-release-package %}). Preserve the original question and earlier response so a later revision or supplier change can be reassessed.

Copy the [blank supplier capability record]({{ "/assets/downloads/supplier-capability/capability-response.md" | relative_url }}). It records no supplier statement or approval until the actual response is attached.
