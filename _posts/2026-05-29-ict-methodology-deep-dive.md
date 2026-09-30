---
title: "ICT Methodology: Select Access, Fault Checks and Evidence"
date: 2026-05-29 14:00:00 +0700
categories: [NPI, Test]
tags: [ict, test-engineering, pcba, dft, fixture, bed-of-nails, flying-probe]
---

> **TL;DR** — Select an in-circuit test method from the board's likely faults, accessible nodes, lot and revision pattern, and the supplier's actual program and fixture proposal. Document the detected fault classes and gaps; no generic percentage proves coverage.

![Conceptual comparison of bed-of-nails and flying-probe ICT; obtain product-specific access, fault coverage, cycle and cost evidence](/assets/img/2026-05-29/ict-methodology-deep-dive.svg)

This is engineering guidance, not a fixture quotation, acceptance limit or test result. The former version included unsupported fixture prices, cycle times, geometry thresholds and volume breakpoints. Those figures are withdrawn. Use the [ICT assessment]({% post_url 2026-05-27-ict-assessment-pattern %}) to record a decision for a real board.

## Start with the fault model

List the faults the build and field use make relevant: shorts, opens, missing or wrong-value passives, reversed parts, incorrect supply rails and faults around programmable devices. For each fault, ask which accessible nodes and measurements can distinguish it from a good board. A measured value can be affected by parallel components, leakage, protection devices and power-state conditions. Demonstrate the measurement on the actual circuit before assigning a pass/fail limit.

ICT does not inherently prove an exact manufacturer part number, component orientation or solder-joint quality. Its ability to detect a fault depends on the board, access, stimulus, measurement method and program. Visual inspection, AOI, boundary scan or FCT may cover a different part of the risk. Keep the methods and their detection claims separate.

| Question | Evidence to collect |
|---|---|
| What is tested? | Net, RefDes or function; fault class; test step and limit source. |
| How is it accessed? | Test point or connector, side, fixture orientation and clearance confirmed on the actual layout. |
| How is it stimulated? | Power state, current limit, isolation and safe sequence. |
| What can cause a false result? | Parallel path, tolerance, fixture contact, temperature or board variant. |
| What is untested? | Inaccessible nets, unobservable faults and checks deferred to another method. |

## Compare implementation routes

**Bed-of-nails:** a custom fixture can contact many designed access points in one setup. Request fixture design, maintenance and change cost for the exact revision. A changed board or test-point location may require fixture work. Confirm probe forces, board support, component clearance and contact repeatability with the supplier.

**Flying probe:** moving probes can avoid a dedicated custom fixture, but programming, access and per-board time still need a current quote and trial. Probe reach, both-side access, tall components and fine features may constrain the test. “No fixture” does not mean no setup cost or full coverage.

**Boundary scan or connector-driven checks:** use only where the devices, pin access, chain, vectors and board configuration support it. Treat these as specific detection methods, not automatic substitutes for physical probing.

For a choice, compare the same fault list, quantity, revisions, expected changes, setup, cycle, operator time, maintenance and retest route across the candidate methods. A break-even calculation needs current quoted inputs; no universal lot-size threshold applies.

## Validate the test before declaring coverage

Inspect the native layout and manufacturing data against the proposed fixture/program. Trial known-good and intentionally controlled fault samples where safe, and record whether the expected fault is detected without unacceptable false rejects. Preserve program revision, fixture ID, limit source, sample configuration, raw results and exceptions. Report **planned** checks separately from **demonstrated** detection. A count of probed nets is access information, not a fault-coverage percentage by itself.

Carry open gaps into the [requirements-to-verification matrix]({% post_url 2026-09-29-requirements-to-verification-traceability %}) and decide whether AOI, FCT, another test or an engineered risk acceptance is needed. A machine PASS applies only to the configured checks and does not authorize shipment.
