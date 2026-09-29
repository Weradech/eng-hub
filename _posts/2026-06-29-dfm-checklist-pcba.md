---
title: "DFM Checklist for PCBA: Check the Actual Build Route"
date: 2026-06-29 10:10:00 +0700
categories: [NPI, DFM]
tags: [dfm, dfa, pcba, smt, pcb, npi, manufacturing]
description: "A process-based PCBA review that ties every concern to the current design, supplier capability and actual assembly route."
---

> **TL;DR** — Review the exact product variant against the intended fabrication, assembly and test route. Record observed conditions with source identity and an actual analysis image. Ask the receiving supplier to confirm process limits; do not treat example dimensions as universal pass/fail rules.

![Proposed DFM review sequence: identify the build, inspect its manufacturing route and record findings with evidence](/assets/img/2026-06-29/dfm-checklist-pcba.svg)
_Workflow illustration only; it is not an image from a board inspection._

This is a proposed review sequence, not a fabrication specification. The customer's controlled requirements, the chosen supplier's documented capability and the agreed build instructions govern the actual decision. A review can identify risk before ordering a board, but its duration and rework cost depend on the project. For the required evidence fields, use [a DFx finding that someone else can verify]({% post_url 2026-09-29-dfx-finding-evidence %}).

## Establish the build baseline

Record product, variant, board revision, panel plan, quantity basis and intended process. Inventory the schematic or netlist, native layout if available, fabrication outputs, BOM with fitted/DNP states, placement file, assembly drawing, stack-up, test requirements and any supplier process notes. Confirm which file is authoritative where instructions overlap. A readable export is not proof that all files describe the same build.

If a source or revision is missing, mark the affected check **NOT REVIEWED**. If the selected supplier has not confirmed a capability that matters, mark **CAPABILITY UNCONFIRMED**. Do not convert either state into a pass.

## Review by operation

| Operation | Questions to resolve | Evidence to retain |
|---|---|---|
| Bare PCB fabrication | Do outline, drill, layer, stack-up, copper and finish instructions match the selected process? Are any features outside a confirmed supplier capability? | Exact fabrication files, viewer/layer image, measured location and supplier response. |
| SMT placement | Can the machine place every fitted part with the stated nozzle, panel support, fiducials and access? Do BOM, placement and drawing population instructions agree? | Placement set comparison, board view, component height/keep-out data and assembler feedback. |
| Paste and reflow | Do land patterns, paste apertures, thermal connections and via treatment suit the components and process? | Native design or paste/copper view, component guidance, stencil proposal and process review. |
| Through-hole or selective solder | Are access, component orientation, adjacent parts and heat exposure compatible with the planned operation? | Board/assembly view, process route and supplier response. |
| Inspection and test | Can the chosen method reach the required features and detect the specified faults? Are polarity markers and test interfaces visible and accessible after assembly? | Inspection/test plan, fixture concept, actual geometry and [requirement-to-method map]({% post_url 2026-09-29-requirements-to-verification-traceability %}). |
| Packaging and handling | Do ESD, moisture, labeling, separation and transport instructions match the product and customer request? | Controlled handling requirements and proposed packing method. |

A generic checklist cannot supply a pass limit for edge clearance, courtyard spacing, fiducial geometry, via-in-pad, probe dimensions or solder paste reductions. Those decisions depend on the board, components, equipment and process. Record the actual requirement or supplier-confirmed limit beside each measurement.

## Write a finding that can be reproduced

For each concern, identify the inspected source and revision or hash, tool version, layer or RefDes, coordinates, measured or visible condition, applicable requirement, affected operation and uncertainty. Attach an **image from the actual analysis** with the location marked. A stock illustration or an unannotated checklist mark is not finding evidence.

Separate **observation** from **possible consequence** and **proposed remedy**. The design owner chooses design changes, the supplier confirms its capability, and the existing release authority decides whether a package may proceed. If the remedy changes a released baseline, use [engineering change control]({% post_url 2026-09-09-engineering-change-control %}).

## Close against the new revision

After a correction, inspect the revised source at the same location. Record the new file identity, image, comparison and remaining issues. A supplier's acceptance of one feature is limited to the stated process and configuration. Reconcile all affected files through the [engineering release package]({% post_url 2026-09-09-engineering-release-package %}) before issuing build instructions.
