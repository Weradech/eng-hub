---
title: "PCBA RFQ Intake: Define the Build Before Pricing It"
date: 2026-09-29 09:00:00 +0700
categories: [NPI, RFQ]
tags: [rfq, pcba, costing, bom, dfx, assumptions]
description: "A lightweight intake method that separates received files, verified scope, quote assumptions and open technical questions."
---

> **TL;DR** — Before calculating a price, establish the product and variant, quantity basis, work scope, available files and unresolved questions. Quote only a defined scope. Keep assumptions visible so a later engineering review can test them.

This is a proposed intake method for a small engineering team. It does not prescribe company prices or approval authority. Continue to the [PCBA costing pattern]({% post_url 2026-05-27-pcba-rfq-costing-pattern %}) only after the quote basis is clear.

Copy the [blank RFQ intake record]({{ "/assets/downloads/rfq-intake/rfq-intake.md" | relative_url }}) for a live request. It starts as `DRAFT` with no decision.

## Record what was actually received

Create one file register for the RFQ. Record the received filename, revision shown inside the file, source, received date and what it is meant to describe. Typical inputs may include a BOM, fabrication data, assembly drawing, placement file, schematic, test requirement and expected volume. A filename or email subject does not establish that two files describe the same build.

| Question | Evidence or explicit open item |
|---|---|
| Which product and population variant is requested? | Customer request and matching controlled identifiers across BOM and drawings. |
| What unit is quoted? | Board, panel, assembly or finished product; boards per panel if relevant. |
| What activity is included? | Fabrication, sourcing, assembly, programming, inspection, test, box build and logistics as applicable. |
| Which quantities and dates matter? | Lot size, forecast basis, prototype versus repeat order, required delivery and quote validity. |
| Which files govern the work? | Actual received revisions, omissions and conflicts. |
| What must the customer decide? | Open questions with owner, requested answer and deadline. |

Separate **received**, **reviewed** and **accepted as quote basis**. A file can be present but still conflict with another file. Record any customer response as a new input; do not silently overwrite the initial RFQ.

## Screen the costs that depend on engineering choices

The first screen identifies drivers, not a final DFx approval. Check part availability and substitution authority, DNP population, placement count, manual operations, soldering process, panel assumptions, special material handling and test scope. If a manufacturing method or test method is undecided, show the cost assumption and the decision needed.

For a DFx concern, record the actual inspected source and revision, location, image from the analysis, observation, consequence and requested disposition. A generic diagram or an unsupported checklist mark is not finding evidence. The [DFM checklist]({% post_url 2026-06-29-dfm-checklist-pcba %}) can guide the screen; the receiving manufacturer must confirm its own process capability.

## Make quote assumptions explicit

Use a short assumption register. Each row needs an assumption, why it matters, who can confirm it and what changes if it is wrong. For example, if the quote assumes that the supplied placement file excludes DNP parts, state that convention and ask the owner to confirm it. Do not count that as a verified population instruction.

| State | Meaning for the quote |
|---|---|
| Confirmed | Supported by the current request or controlled evidence. |
| Assumed | Used for pricing, with a stated impact and pending confirmation. |
| Excluded | Outside the offered scope and visible in the quote. |
| Blocked | Prevents a defensible price or commitment for the affected scope. |

Keep internal cost calculation, commercial margin and the customer quotation distinct. Unit price, one-time charges, quantity, lead time, validity, exclusions and change conditions must use the organization's current commercial rules. Published example rates are not a supplier quotation or authority to set a customer price.

## Hand over a quote baseline

When the quote is issued, retain the file register, assumptions, estimate revision, quoted scope, customer clarifications and open risks. If the customer changes the BOM, quantity or test scope, revise the estimate and identify what changed. The accepted quote basis becomes an input to the [engineering release package]({% post_url 2026-09-09-engineering-release-package %}); it does not itself authorize manufacturing.
