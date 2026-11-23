---
title: Start Here
icon: fas fa-compass
order: 1
---

## Start with the decision you need to make

This hub contains working methods for a small, high-mix, low-volume engineering team. Choose the work in front of you. Each path names the record to create and the decision it supports. The articles propose methods; they do not grant company approval or replace a customer specification.

## Respond to a PCBA RFQ

Read [RFQ intake and quote boundary]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}), then the [costing pattern]({% post_url 2026-05-27-pcba-rfq-costing-pattern %}). Record missing inputs, quote assumptions and the scope being priced.

## Review a PCB or PCBA build package

Use the [DFM checklist]({% post_url 2026-06-29-dfm-checklist-pcba %}) and [DFx finding evidence method]({% post_url 2026-09-29-dfx-finding-evidence %}), then reconcile the [engineering release package]({% post_url 2026-09-09-engineering-release-package %}). Capture analysis images for findings and use the [release records]({{ "/templates/" | relative_url }}).

For a process limit the team cannot verify, request an answer from the selected manufacturer with the [supplier capability handoff]({% post_url 2026-09-29-supplier-capability-handoff %}).

See a [dated check of public supplier and lab capability pages]({% post_url 2026-09-29-public-provider-capability-check %}) for what those pages can establish before a project-specific response arrives.

## Release a PCB to fabrication

Follow the [gated PCB build]({% post_url 2026-11-22-gated-pcb-build-to-fab-release %}). Open the customer's own design files before redesigning, record a reviewer decision after each step, and generate fabrication files only after an explicit release decision on the reviewed file set.

## Change a released component or instruction

Use [engineering change control]({% post_url 2026-09-09-engineering-change-control %}). Record impact, verification, material disposition and effectivity before implementation.

When the change replaces a component, check the [exact MPN of the replacement]({% post_url 2026-11-22-exact-mpn-replacement-check %}) against its own datasheet (pins, package, the electrical table for that variant and how it will be bought) before it enters the BOM. The approval and effectivity decision then returns to change control.

The [public ESP32-PoE revision case]({% post_url 2026-09-29-public-revision-case-poe-class %}) shows how reported field feedback and a schematic change can be traced without inventing test or release evidence.

## Plan ICT or FCT

Start with [requirements-to-verification traceability]({% post_url 2026-09-29-requirements-to-verification-traceability %}), then select checks with the [ICT assessment]({% post_url 2026-05-27-ict-assessment-pattern %}) and [FCT method]({% post_url 2026-05-29-fct-methodology-deep-dive %}). Record what each method can detect and its coverage gaps.

## Request a pre-compliance lab test

Use the [test intake and evidence method]({% post_url 2026-09-29-precompliance-test-intake %}). Define the exact unit, method, conditions and report scope before testing; keep exploratory findings separate from formal certification claims.

## Respond to a traceability gap on the line

Use the [traceability hold scope method]({% post_url 2026-11-22-traceability-gap-hold-scope %}) when a unit's record is missing or contradicts the physical product. Describe the gap without guessing its cause, hold what cannot yet be excluded, and release units only against criteria written before evaluating evidence for release. The [DMC traceability pattern]({% post_url 2026-08-26-shopfloor-dmc-family-traceability %}) shows how those records are created.

## Decide what follows a pilot

Use the [pilot build exit review]({% post_url 2026-09-09-pilot-build-exit-review %}). Reconcile units and issues, then request a decision for a named next activity.

## Design or repair a WMS/MES integration

Start with [financial versus operational ownership]({% post_url 2026-05-27-financial-core-vs-operational-core %}), then [stock reconciliation]({% post_url 2026-06-13-reconciliation-without-false-alarms %}) and [resilient ERP integration]({% post_url 2026-08-26-resilient-erp-integration-patterns %}). Identify the authoritative source, transaction boundary and failure recovery before changing data.

## Release a controlled document revision

Use the [document revision release workflow]({% post_url 2026-11-22-release-a-controlled-document-revision %}) and the [SOP hierarchy]({% post_url 2026-06-29-sop-hierarchy %}). Before release, confirm the change request, the closed review comments, the approver, the effective date and every form that refers to the procedure. Record which revision it replaces. For keeping the document set itself as version-controlled files, see [document-as-code for an ISO 56001 document set]({% post_url 2026-08-26-document-as-code-for-iso56001 %}).

## Write a controlled procedure or investigate an incident

Use the [SOP hierarchy]({% post_url 2026-06-29-sop-hierarchy %}) and [SOP revamp method]({% post_url 2026-05-27-sop-revamp-playbook %}) for documentation. For a software symptom, start with [observations before hypotheses]({% post_url 2026-05-27-rca-for-software-symptoms-before-hypothesis %}). Keep the controlled source, actual authority and evidence of effectiveness visible.

## See the whole path

[A worked NPI thread]({% post_url 2026-09-29-one-build-from-rfq-to-pilot %}) connects RFQ, release, pilot and the next decision using fictional records. It shows where work stops when evidence or authority is missing.

The [template index]({{ "/templates/" | relative_url }}) lists every downloadable working record, its inputs and its output. Copy only the records the current scope needs. Leave approval fields empty until a real authorized decision exists.

## Evidence rule

An available file is not a reviewed file. A passed machine check is not engineering closure. A reviewed package is not production or shipment authorization. For a DFx finding, attach an image from the actual analysis and identify the source, file revision, location, observation and decision needed. Do not use an illustration as inspection evidence.
