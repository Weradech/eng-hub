---
title: "A Public Revision Case: PoE Classification Changed After Field Feedback"
date: 2026-09-29 15:00:00 +0700
categories: [NPI, Process]
tags: [public-case, engineering-change, poe, configuration, evidence]
description: "A source-pinned case study of an ESP32-PoE revision change, separating the maker's field report from what is visible in its published schematic."
---

> **TL;DR** — A change can be electrically plausible and still affect the system in which customers use the product. This public case links the reported field effect, the revised schematic and the verification questions that a release review should answer.

This is an analysis of **public OLIMEX ESP32-PoE files**, not a Synergy customer project. The repository was inspected at commit [`deb38a3`](https://github.com/OLIMEX/ESP32-POE/tree/deb38a377d50eadb04ddc7118a17a5ae58b1ee53) on 29 September 2026. Revisions M and M1 are historical examples; the repository also lists a later M2 revision. No private customer details or test results are used here.

## What the source says

The maker's [revision history](https://github.com/OLIMEX/ESP32-POE/blob/deb38a377d50eadb04ddc7118a17a5ae58b1ee53/HARDWARE/Revision-changes.txt) says revision M changed the PoE class from 0 to 4. It says M1 reversed that change after complaints from customers using several devices on one power source because the higher reservation caused a power-budget problem. Those are **OLIMEX's reported history and explanation**. The public files do not give a customer count, test report, failure rate or measured site power budget.

## What we checked in the published design

We inspected the two [schematic PDFs for M](https://github.com/OLIMEX/ESP32-POE/blob/deb38a377d50eadb04ddc7118a17a5ae58b1ee53/HARDWARE/ESP32-PoE-hardware-revision-M/ESP32-PoE_Rev_M.pdf) and [M1](https://github.com/OLIMEX/ESP32-POE/blob/deb38a377d50eadb04ddc7118a17a5ae58b1ee53/HARDWARE/ESP32-PoE-hardware-revision-M1/ESP32-PoE_Rev_M1.pdf) at that same commit. In the visible U9 classification circuit, R54 is marked `66.5R/1%/125mW/R0805` in M and `NA(66.5R/1%/125mW/R0805)` in M1. `NA` is the schematic's non-assembly designation. This file comparison supports that the classification-area population instruction changed; it does **not** by itself establish how every physical M1 board was populated or tested.

![Crop of the published M schematic showing R54 as populated near U9](/assets/img/2026-09-29/olimex-poe-m-classification.png)

*Revision M: R54 is shown without the NA prefix.*

![Crop of the published M1 schematic showing R54 marked NA near U9](/assets/img/2026-09-29/olimex-poe-m1-classification.png)

*Revision M1: the same position is marked NA. The `Class4_EN1 Closed` label is still visible in both drawings, so the PDF alone should not be used to infer a complete build configuration.*

These images are **cropped** from OLIMEX's schematic PDFs, with no circuit edits. The repository supplies an [Apache-2.0 license]({{ "/assets/img/2026-09-29/OLIMEX-LICENSE.txt" | relative_url }}); the crops retain source credit and show only the classification area. The downloaded PDF SHA-256 values were `23fcd89ed24be5e54a9c0061f1e320dd0ae3df1d817a95a23637dab98f61e3b0` (M) and `0eaa8fc6db0c95aab29253584c558960fe8ed917fbb786fbae4f92c0496a4917` (M1). These hashes identify the inspected files; they are not a release signature.

## A bounded NPI response to the same kind of change

For a product under our control, record the reported use case and reproduce the power-source arrangement before selecting a remedy. Compare the old and proposed configuration across schematic, BOM, assembly instruction, firmware where relevant and test method. Ask whether the system-level power budget and mixed-revision use cases were included in verification. Capture the result, open gaps and affected serial or lot range, then request a decision for the named next activity. The [change record]({% post_url 2026-09-09-engineering-change-control %}) and [requirements-to-verification matrix]({% post_url 2026-09-29-requirements-to-verification-traceability %}) provide the two places to preserve that chain.

This case illustrates a **review question**, not a DFx defect verdict on OLIMEX hardware or an approval of either revision. No bench test, board DRC, physical inspection, customer-return data or production-release record was available for this analysis.
