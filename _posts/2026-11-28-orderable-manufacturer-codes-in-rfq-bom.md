---
title: "Verify Orderable Manufacturer Codes in an RFQ BOM"
date: 2026-11-28 09:00:00 +0700
categories: [NPI, RFQ]
tags: [rfq, bom, mpn, ordering-information]
description: "Check whether an RFQ BOM line identifies a complete manufacturer order code, then record the evidence or the open question."
---

> **TL;DR** — Keep the description, supplier SKU and manufacturer order code in separate fields. Verify the complete code against the manufacturer's ordering information. Leave the identity unresolved when the intended variant or suffix is unclear.

## Read the three fields separately

The RFQ intake question is whether the BOM line identifies one specific manufacturer order code.

- **Description.** The requested characteristics, such as function and package. Several parts can fit the same description.
- **Supplier SKU.** A listing in a supplier's catalogue. Use the listing as a lead to the manufacturer and its order code.
- **Manufacturer and complete order code.** The manufacturer and the full manufacturer part number (MPN) used to order the variant, including any required suffix.

## Flag suspect entries for review

Flag blanks, placeholders such as `TBD`, and entries that consist only of a value and package or a family name. A trailing ellipsis or a code ending before the suffix shown on a supplier listing can signal truncation. Use those patterns to prioritize review. Apply the source check below to entries that look complete as well. Length, punctuation and naming conventions vary by manufacturer.

## Confirm the exact order code

1. Preserve the submitted entry. Find the manufacturer's product page, ordering table or package addendum. Use a supplier listing to locate a candidate code if needed.
2. Compare the full string with the manufacturer's ordering information, including letters, digits and separators. Check what the suffix selects for that family. It may specify the package, grade or shipping format. If several listed codes fit the description, ask which variant is intended and keep the row unresolved until there is an answer.
3. Record the manufacturer and confirmed code beside the original entry. Capture the source link or document title, revision if available, exact row or section, and date checked. If the source is missing or conflicts with the request, mark the line unresolved and record the discrepancy and who can answer the next question. Keep any candidate separate from the confirmed order code.

## Example intake row

The manufacturer, codes and ordering table in this example are fictional.

- **Description.** Surface-mount signal switch.
- **Supplier SKU.** `DEMO-SHOP-417`.
- **Submitted manufacturer field.** Example Components, `DEMO-SW17`.
- **Ordering evidence.** The table lists `DEMO-SW17-TR` for tape and reel and `DEMO-SW17-TB` for tube.
- **Status.** Unresolved. Ask which shipping format the RFQ requires.

After clarification, record the selected full code, the ordering-table row and the source of the clarification. Mark the orderable identity confirmed once those records agree.

## Track qualification and purchasing separately

A confirmed identity tells the RFQ team which part to reference in a supplier inquiry. Keep separate records for design suitability and qualification, current availability through the intended channel, and authorization to buy.

## Related Posts

- [BOM Lifecycle]({% post_url 2026-05-27-bom-lifecycle %}) follows the BOM from RFQ through production.
- [Exact MPN Replacement Check]({% post_url 2026-11-22-exact-mpn-replacement-check %}) covers design checks when a candidate may replace a released part.
- [PCBA RFQ Intake]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}) covers the build scope and inputs needed for a quote.
