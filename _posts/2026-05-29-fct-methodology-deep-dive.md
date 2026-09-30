---
title: "FCT Methodology: Turn Product Requirements into Repeatable Checks"
date: 2026-05-29 14:30:00 +0700
categories: [NPI, Test]
tags: [fct, functional-test, test-engineering, pcba, firmware, calibration]
---

> **TL;DR** — Functional test applies defined inputs and loads to a known product configuration, observes outputs and compares them with controlled requirements. Keep the program, fixture, limits and raw result linked to each tested unit.

![Conceptual FCT stages selected from product requirements; results and release authority remain separate](/assets/img/2026-05-29/fct-methodology-deep-dive.svg)

This is a planning method, not a test report or quotation. The former version listed unsupported fixture prices, test times, coverage percentages and default limits. Those figures are withdrawn. Define each product's test scope with the [requirements-to-verification matrix]({% post_url 2026-09-29-requirements-to-verification-traceability %}).

## Define the equipment under test

Identify the board and assembly revision, fitted options, firmware and configuration, serial number, supply, load, cables and operating mode. A test result belongs to that configuration. If a firmware or component change affects a check, assess whether previous evidence remains applicable.

Select checks from product requirements and failure risks. They may include safe power-up and current draw, rail behavior, communications, sensors, actuators, timing, calibration and end-product interaction. “LED lights” or “relay clicks” is not a substitute for a quantitative requirement when the application needs a measured value. Equally, a precise measurement without a controlled limit does not yield a valid PASS/FAIL conclusion.

| Test definition | Record before running |
|---|---|
| Requirement | Source ID, revision, limit or expected behavior and allowed conditions. |
| Stimulus | Input, load, sequence, duration and operating state. |
| Measurement | Instrument, fixture, software, location and calibration status where relevant. |
| Decision | Comparison rule, uncertainty where it matters, retries and abort handling. |
| Trace | Unit ID, program revision, raw result, timestamp, operator and deviations. |

## Build a safe and repeatable station

Set power limits and an abort path before testing an unproven board. Confirm connector mating, polarity, contact pressure and fixture protection. Separate programming, calibration and functional checks in the record even if one station performs all three. A flashed firmware version is not proof that a functional step passed. A calibration adjustment needs its before/after value, reference and acceptance rule.

Choose a pogo fixture, product connector, manual rig or system simulator from the access and load required by the product. Request current setup, cycle, maintenance and operator-time data for the actual station. Multiple simultaneous channels add shared-instrument and cross-talk questions; demonstrate them rather than assuming throughput scales with channel count.

## Prove the configured coverage

Run a known-good reference and controlled fault cases where safe. Check that the station rejects the relevant faults, handles an interrupted test, prevents a partial run from becoming PASS and retains diagnostic detail. Compare results across fixtures and shifts only after confirming product, firmware, program, limits and sample mix. Record false rejects and escapes that can be demonstrated.

List what FCT does not observe: inaccessible functions, operating modes, environmental conditions, lifetime behavior or interfaces left to another qualification activity. Map those gaps to the verification plan. ICT can complement FCT, but neither method's PASS grants production or shipment authorization; see the [ICT method]({% post_url 2026-05-29-ict-methodology-deep-dive %}) and [pilot exit review]({% post_url 2026-09-09-pilot-build-exit-review %}).
