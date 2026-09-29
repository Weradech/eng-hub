---
title: "ICT Assessment Pattern: Decide from Faults, Access and Actual Cost"
date: 2026-05-27 12:00:00 +0700
categories: [NPI, Test]
tags: [ict, dft, test-engineering, npi, pcba, nre]
description: "A board-specific ICT decision using required faults, physical probe access, supplier capability and current offers."
---

> **TL;DR** — Count physical test access, map the faults it can detect, and compare actual ICT, flying-probe and functional-test options. A test-point count or nine-item score cannot establish fault coverage or justify a fixture by itself.

![ICT assessment sequence from requirements and board access to detectable faults and current cost comparison](/assets/img/2026-05-27/ict-assessment-pattern.svg)
_Workflow illustration only; no actual board or test result is represented._

This is a proposed assessment method for NPI. No universal test-point diameter, pitch, coverage percentage, score threshold, fixture price or break-even volume is supplied. Obtain the limits from the selected tester and board manufacturer, the expected build pattern and the product's requirements. See the [ICT method overview]({% post_url 2026-05-29-ict-methodology-deep-dive %}) for concepts; its historical numeric examples are not current quotes.

## 1. Define the fault question

Start with the controlled product and variant, intended build quantity, revision frequency, likely defect classes and customer test obligations. Use the [requirements-to-verification matrix]({% post_url 2026-09-29-requirements-to-verification-traceability %}) to name the condition and acceptance criterion for each relevant check. A requirement that needs firmware, timing or real load behavior may need FCT even if ICT access is excellent.

## 2. Inspect the actual board and test route

Gather the schematic or netlist, native PCB or fabrication data, BOM, assembly drawing and placement file. Record source revisions. Mark accessible test pads, vias, connectors and rails on the exact assembly configuration. Check the chosen fixture side, component obstructions, board support, keep-outs, panel handling and electrical safety. Ask the test supplier to confirm probe geometry, instrument capability and program assumptions.

A via is not automatically a usable probe point. A labelled test point is not automatically accessible in a loaded fixture. Count **unique nodes and fault opportunities**, not just visible pads. Physical node access is a useful input; it is not the same as detection coverage for opens, shorts, wrong values, polarity or latent behavior.

## 3. Map detection and gaps

| Fault or requirement | Candidate method | What must be confirmed |
|---|---|---|
| Short or open | ICT, flying probe or another electrical check | Accessible endpoints, safe stimulus and a program that distinguishes the fault. |
| Missing or wrong component | Electrical measurement, AOI or combined method | Whether parallel paths or visual similarity make the result ambiguous. |
| Polarity or orientation | AOI, ICT or FCT | Whether the selected method observes the error on this package and board. |
| Firmware or system behavior | FCT or system test | Exact input, load, limits and configured result. |

Record `COVERED WITH EVIDENCE`, `PLANNED`, `GAP` or `N/A WITH REASON` per fault. Do not multiply an accessible-node ratio into a product-wide defect-detection percentage.

## 4. Compare current offers and lifecycle cost

Ask suppliers for fixture and program NRE, per-board cycle and fee, setup, maintenance, revision changes, expected retests and lead time. Compare those against the customer's expected lot pattern and the detection value of each method. A flying-probe route still has programming and cycle costs. A bed-of-nails fixture may need changes after a PCB revision. Calculate break-even from these actual inputs; do not import an old threshold.

## 5. Deliver a bounded recommendation

The assessment should state the selected configuration, accessible nodes, mapped fault coverage, untested requirements, supplier confirmation, cost basis and the recommended ICT/FCT/AOI combination. Attach board images and source identities for any DFx finding. If a layout change is proposed, use [the DFx evidence method]({% post_url 2026-09-29-dfx-finding-evidence %}) and route a released-baseline change through the applicable control process. A recommendation is not a customer approval or a passed production test.
