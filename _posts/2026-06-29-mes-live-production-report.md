---
title: "Building a Production Report from MES Traceability Data"
date: 2026-06-29 11:10:00 +0700
categories: [System, MES]
tags: [mes, reporting, api, production, traceability, html]
---

> **TL;DR** — Freeze a defined MES data extract, reconcile its unit and event counts, then render a dated report whose filters, formulas and exclusions are visible. A chart is a view of the extract, not evidence that the process improved or that a lot is released.

This is a proposed reporting method. The endpoint names and figures below are **illustrative**, not a claim about a deployed MES, a production lot or current performance. Do not insert actual customer or production data into a public demonstration.

## Establish the source and denominator

Identify the product, variant, lot, serial range, time zone, shift boundary, test-program revision and extraction time. Preserve a read-only source export or query receipt. Count unique units separately from test attempts: a retest must not silently become a new unit. State whether aborted tests, diagnostic runs, missing serials and rework attempts are included.

For each unit, retain the first attempt, latest attempt and final disposition as separate facts. First-pass yield needs a stated denominator of eligible units and a stated rule for what counts as a first attempt. A PASS on the latest attempt does not erase an earlier failure. A box-build match establishes only that the scanned identifiers matched; it does not approve shipment.

| Report element | Required source or rule |
|---|---|
| Units and attempts | Unique serial count and test-event count, with exclusions listed. |
| First-pass result | First eligible attempt per unit; explain missing or aborted attempts. |
| Step failures | Test-program step ID and revision, with counts per attempt or per unit explicitly chosen. |
| Jig comparison | Comparable product, program, time window and sample size for each jig. |
| Retest distribution | Number of attempts per unit, linked to final disposition. |
| Trend | Local time-zone boundary, denominator and configuration changes marked on the plot. |

## Investigate a pattern without declaring its cause

Suppose a **synthetic training dataset** has 100 eligible units on each of two jigs, with 73 first-pass units on one jig and 81 on the other. That difference is a question: inspect unit mix, sample size, fixture contacts, cables, test-program revision and operator sequence. It is not evidence that a jig has drifted. Likewise, several communication steps failing together narrows the investigation but does not prove a firmware fault; a shared fixture or configuration error may produce the same pattern.

Choose an alert rule from actual process risk and baseline data. A threshold, comparison period or control limit is not universal. An alert should point to the affected unit set and the raw events so an engineer can reproduce the calculation and investigate. Record the finding, owner and disposition before calling the issue closed.

## Publish a controlled snapshot

Generate the HTML or PDF from the frozen export, with an extract timestamp and source identity in the report. Keep the data and report together under the applicable access controls. If an HTML file relies on a CDN or a live API call, it is **not** an offline standalone snapshot. For offline use, bundle required assets and embed or package the approved data extract; test the actual opened artifact without network access.

Before sharing, check that serial numbers, customer names, MAC addresses, credentials and raw measurements are authorized for that audience. A report with embedded production data remains sensitive even when it has no login screen. Reconcile displayed totals to the extract and keep the report revision linked to the extract hash. A reviewed report still needs a separate [pilot or lot decision]({% post_url 2026-09-09-pilot-build-exit-review %}) under the responsible authority.
