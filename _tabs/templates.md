---
title: Templates
icon: fas fa-file-alt
order: 2
---

## Working records

Use these files with the linked method. They are starting records, not approved company forms. Set the applicable authority and document controls for your organization. **Blank** files have no decision or approval. **Synthetic** files are training examples and must not be imported into live systems.

## PCBA RFQ intake

**Input:** request, variant, quantity basis, received files, operations and unknowns. **Output:** quote basis, assumptions and customer questions. Copy the [blank RFQ intake record]({{ "/assets/downloads/rfq-intake/rfq-intake.md" | relative_url }}) and follow the [method]({% post_url 2026-09-29-rfq-intake-and-quote-boundary %}).

## DFx finding

**Input:** inspected source, tool, location, applicable rule and an image from the actual analysis. **Output:** reproducible finding, owner, disposition and verification evidence. Copy the [blank DFx issue sheet]({{ "/assets/downloads/dfx-finding/issue-sheet.md" | relative_url }}) and follow the [method]({% post_url 2026-09-29-dfx-finding-evidence %}).

## Engineering release package

**Input:** product, variant, intended activity, source revisions, exports, supplier conventions and checks. **Output:** file inventory, discrepancies and scoped release decision. Use the [release checklist]({{ "/assets/downloads/engineering-release-package/release-checklist.md" | relative_url }}), [Markdown manifest]({{ "/assets/downloads/engineering-release-package/release-manifest.md" | relative_url }}) or [JSON alternative]({{ "/assets/downloads/engineering-release-package/release-manifest.json" | relative_url }}). Read the [guide]({{ "/assets/downloads/engineering-release-package/template-guide.md" | relative_url }}).

## Engineering change control

**Input:** released baseline, proposed change, affected requirements, parts, stock and builds. **Output:** impact, verification, disposition, effectivity and decision. Use the [impact assessment]({{ "/assets/downloads/engineering-change-control/change-impact-assessment.md" | relative_url }}), [JSON record]({{ "/assets/downloads/engineering-change-control/change-record.json" | relative_url }}) and [guide]({{ "/assets/downloads/engineering-change-control/template-guide.md" | relative_url }}). The [worked example]({{ "/assets/downloads/engineering-change-control/worked-example.md" | relative_url }}) is synthetic.

## Requirements and verification

**Input:** controlled requirements, limits, configuration, procedure and available evidence. **Output:** requirement-to-method map, results and visible coverage gaps. Copy the [blank verification matrix]({{ "/assets/downloads/verification/requirements-verification-matrix.md" | relative_url }}) and follow the [method]({% post_url 2026-09-29-requirements-to-verification-traceability %}).

## Pilot build exit review

**Input:** actual configuration, unique unit histories, test boundary, deviations and open issues. **Output:** reconciled outcomes, action log and scoped next-build review. Use the [pilot review]({{ "/assets/downloads/pilot-build-exit-review/pilot-build-review.md" | relative_url }}), [open-issue log]({{ "/assets/downloads/pilot-build-exit-review/open-issue-log.md" | relative_url }}) and [guide]({{ "/assets/downloads/pilot-build-exit-review/template-guide.md" | relative_url }}). The [20-unit dataset]({{ "/assets/downloads/pilot-build-exit-review/demo-lot.json" | relative_url }}) is synthetic.

For a quick outline covering costing, RCA, gates or work instructions, use the [NPI templates toolkit]({% post_url 2026-05-27-npi-engineering-templates-toolkit %}) and adapt it to the current scope.

## Pick one authoritative record

The release manifest has Markdown and JSON versions. Use one as the controlled source; regenerate or reconcile the other if both are kept. A valid JSON file proves only that its syntax can be parsed. Empty arrays and `PENDING` fields do not mean a review passed.

Use the [Start Here]({{ "/start-here/" | relative_url }}) paths to select the method. The [worked NPI thread]({% post_url 2026-09-29-one-build-from-rfq-to-pilot %}) shows how these records connect.
