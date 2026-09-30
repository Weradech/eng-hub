---
title: "PCBA RFQ Costing: Build a Traceable Quote Basis"
date: 2026-05-27 09:00:00 +0700
categories: [NPI, RFQ]
tags: [pcba, costing, rfq, bom, npi, ems]
---

> **TL;DR** — Price the exact product revision, quantity and manufacturing scope from current evidence. Separate recurring cost, setup/NRE, assumptions, commercial policy and customer-facing terms. A formula is only as sound as its inputs and authority.

This is an RFQ method, not a Synergy price list or approved margin policy. The former article's placement rates, setup charges, margin bands and worked totals had no current supplier offer or approved cost-model source. They have been removed. Use [RFQ intake]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}) to close the scope before calculating.

## Establish the priced configuration

Identify the customer request, product and revision, BOM and AVL, fitted options, drawing and fabrication package, order quantity and delivery schedule. Reconcile BOM quantities and DNP positions to assembly data. Record approved substitutions and any material that the customer supplies. A missing MPN, test definition or material specification is an RFQ question, not a price to invent.

The quote basis should name the intended build route: PCB fabrication, procurement, SMT, through-hole, hand work, inspection, programming, ICT/FCT, calibration, coating, box build, packaging and shipping as applicable. For each operation record whether it is included, excluded, optional or awaiting a supplier answer. Confirm process capabilities with the selected supplier for the exact build rather than adopting a generic rate card.

## Keep cost lines and units visible

| Cost group | Evidence needed |
|---|---|
| Material | Current supplier offer, order quantity/MOQ, yield or attrition basis, currency and validity. |
| Bare PCB | Controlled stack-up and build spec, quantity and supplier quotation. |
| Assembly | Routing, placements, special operations, setup and supplier or internal labor basis. |
| Test | Test method, coverage, fixture/programming, cycle and retest basis. |
| One-time work | Stencil, tooling, fixture, engineering and qualification scope, priced separately from recurring units. |
| Commercial terms | Approved overhead, margin or markup method, taxes, freight, exchange-rate basis and validity. |

Use one consistent unit basis: cost per ordered board, cost per good board or cost per lot. Show any assumed scrap or retest allowance explicitly so it is not counted twice. Keep source currency and date with each offer. Do not mix margin and markup: a gross-margin formula and a cost-markup formula produce different prices for the same cost.

## Reconcile the proposed quote

Calculate the recurring unit cost from the current approved inputs, then apply the organization's approved commercial policy. Carry one-time charges as distinct lines unless the approved quote method explicitly amortizes them over a named quantity. Check quantity breaks, lead time, capacity, tooling ownership, customer-supplied material and quote validity. Finance or the designated commercial owner must decide how much cost detail the customer-facing offer shows; there is no universal “lump sum only” rule.

Before release, compare the quotation against its own source list: every priced operation has an input, every exclusion is visible, and each unresolved assumption has an owner and a stated impact. An internal spreadsheet that balances mathematically is not proof that the customer scope or supplier capability is correct.
