---
title: "Sizing a PCB from Gerbers for an RFQ: What the Files Can Establish"
date: 2026-06-29 10:00:00 +0700
categories: [NPI, PCB]
tags: [pcb, gerber, costing, rfq, npi, eda]
---

> **TL;DR** — Gerbers can help measure the board outline and identify the plotted copper layers. They do not, by themselves, establish substrate, stack-up, copper weight, finish, panel yield or a supplier price. Put those missing inputs on the RFQ question list.

This is a preparation method, not a supplier capability statement or a quotation. The earlier version of this article included unsourced panel prices and a supposed 2.5× underquote case. That case has no attached source package or quote, so it is withdrawn rather than presented as a real outcome.

## Identify the actual outline

File suffixes such as `.GKO`, `.GM1` and `Edge_Cuts.gbr` are useful clues, not a guarantee. Open the complete package in a Gerber viewer, map each file to its intended layer and inspect the perimeter, slots, cutouts and drill data. Check units, coordinate format, zero suppression and image polarity before measuring. If the outline is incomplete or ambiguous, ask for the fabrication drawing or native CAD source.

The outer **bounding box** gives maximum X and Y extent. It is not necessarily the finished area or a feasible panel nest: an irregular outline, internal cutout, tab, tooling rail or routing gap changes material use and panel count. Record the source filename, revision, measured dimensions, method and reviewer. A screenshot from the inspected file should show the outline and measurement markers.

## Keep material and process as open requirements

One plotted copper layer does not prove an aluminium-core board; one- or two-layer FR-4 and other constructions may fit the same visible Gerber pattern. An absent drill file may mean a missing export. A mounting-hole drill file does not identify the substrate. Do not infer dielectric material, thickness, copper weight, thermal performance, surface finish or plated-hole construction from filenames alone.

Ask the customer or design owner for the controlled fabrication drawing or stack-up, plus material, finished thickness, copper weights, finish, mask, legend, hole and slot requirements, tolerances, quantity and intended assembly route. Record any proposed alternative as an assumption requiring confirmation. If the source files conflict, keep the quote on hold or price clearly separated options with the uncertainty stated.

## Request a build-specific quote

For a rough internal comparison, compute the board bounding box and test possible orientations within a **supplier-confirmed usable panel**, including rails, spacing, breakaway method and process restrictions. Ask the supplier to confirm the actual panelization, yield, setup charges, unit price, lead time and validity for the exact revision and order quantity. Do not treat a generic A4 panel, public capability limit or historical rate as a current offer.

| RFQ input | Status to record |
|---|---|
| Outline and drill set | Source files, revision, measured geometry and any ambiguous features. |
| Build specification | Controlled material, stack-up, copper, finish, tolerances and special processes. |
| Commercial basis | Quantity breaks, setup/NRE, panelization, scrap basis, currency, freight and validity. |
| Open questions | Owner, requested evidence, proposed assumption and quote impact. |

Use the [RFQ intake record]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}) to track missing inputs and the [supplier capability handoff]({% post_url 2026-09-29-supplier-capability-handoff %}) for a specific process question. A measured outline makes the request more precise; only the controlled build specification and supplier response can close the costing basis.
