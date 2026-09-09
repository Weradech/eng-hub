---
title: "Engineering Release Package: Are These Files Ready to Build?"
date: 2026-09-09 09:00:00 +0700
categories: [NPI, Process]
tags: [release, bom, gerber, pcba, configuration-management, manufacturing]
description: "A practical release review for small NPI teams, with a fictional BOM/PnP mismatch and editable checklist and manifest templates."
---

> **TL;DR** — A build package needs a defined product variant, traceable file revisions, consistent manufacturing instructions, and a recorded release decision. This article provides a review sequence and editable templates for checking those conditions before a package reaches the factory.

![Release package review: identify the build, reconcile the files, review evidence, and record a scoped GO or HOLD decision](/assets/img/2026-09-09/engineering-release-package.svg)
_A proposed workflow for a small NPI team. The worked example below uses fictional data; it does not describe an actual customer release._

## The Problem: Every File Opens, but the Build Is Ambiguous

Imagine receiving a fabrication archive, a BOM, a component placement file, and an assembly drawing. The archive opens. The spreadsheet looks complete. The drawing has a revision in its title block.

The BOM says that R12 is not fitted for the requested variant. The placement file still includes R12, with no population flag, and the assembly drawing calls for it to be fitted. Each file is readable, but the package gives conflicting build instructions.

A useful release review answers four questions:

1. Which product, variant, and build scope does this package authorize?
2. Which exact files belong to that configuration?
3. Do the files agree where their instructions overlap?
4. Who made the release decision, using which evidence and conditions?

This complements the existing [BOM lifecycle]({% post_url 2026-05-27-bom-lifecycle %}) and [NPI Stage Gate]({% post_url 2026-05-27-npi-stage-gate %}) articles. It proposes a working method, rather than a company approval policy or a certification requirement.

## Start with the Build Scope

Write the scope before collecting signatures. A package for bare-board fabrication and a package for programmed, tested assemblies need different evidence.

| Intended activity | Information to resolve before that activity |
|---|---|
| Bare PCB fabrication | Board identity, fabrication data, drilling, outline, stack-up and fabrication requirements agreed with the supplier |
| PCBA assembly | Fabrication baseline plus population variant, BOM, placement instructions, assembly drawing and relevant process requirements |
| Programming and functional test | Assembly baseline plus firmware identity, programming procedure, test method, limits and required equipment |
| Finished-product shipment | Applicable build/test evidence plus labeling, packaging, traceability and customer acceptance requirements |

Record applicability explicitly. For example, a board with no programmable device can mark firmware as **N/A with a reason**. Missing firmware for an assembly that requires programming remains an open item.

Use the customer's agreed deliverables and the receiving manufacturer's requirements to tailor this list. A netlist, stencil specification or panel drawing may be required for a particular workflow. Do not silently assume that a missing item is optional.

## Build a Manifest of the Actual Configuration

The manifest is the index of the release. Keep one row per controlled file, or one row for an archive with a separately recorded inventory of its contents.

| Field | What to record |
|---|---|
| Package ID and revision | Identity of this issued package |
| Product and variant | Exact configuration to build |
| Build scope and effectivity | Authorized activity and applicable order, lot or serial range |
| File path and role | Package-relative path and purpose of each file |
| Document revision | Revision recorded in the file or its controlled source |
| SHA-256 | Hash calculated from the exact file being issued |
| Source baseline | Design revision, controlled export record or source commit, where available |
| Review evidence | Check record, comparison report, image or supplier response |
| Decision record | Decision, authorized person, date and conditions |

**Different document revision numbers can belong to the same valid package.** PCB Rev C, BOM Rev 07 and assembly drawing Rev D can be a consistent configuration when their compatibility is established and recorded. Matching filenames or revision suffixes alone cannot establish that compatibility.

Calculate hashes after final export. A SHA-256 match helps establish that the recipient has the same bytes; it does not establish technical correctness or approval. Keep the manifest outside its own file-hash inventory to avoid a self-reference. If sending a ZIP, record its hash in the transmittal after creating it.

The downloadable manifest starts as `DRAFT`, with a `PENDING` decision and empty approval fields. JSON syntax validity has no effect on those fields.

## Reconcile the Files in Four Passes

### 1. Identity and revision mapping

Compare the product, variant and board identity across the manifest, BOM, drawings and export source. Record how each document revision maps to the build baseline. Where the files disagree, obtain a resolved instruction from the responsible design owner and preserve that evidence.

If native design files are outside the handoff scope, record which controlled export or supplier/customer confirmation establishes the baseline. Do not fill that gap by inferring a revision from a filename date.

### 2. BOM and placement coverage

Normalize reference designators, expand grouped references, and check for duplicates. Separate fitted machine-placed components, fitted manual components, DNP items, and non-component features such as fiducials.

Then compare sets of references and their instructions. Equal totals can still hide a swapped or missing reference. Quantity must also agree with the expanded reference list and the declared unit of manufacture: one board or a panel.

KiCad's placement export supports configurable exclusions for DNP and through-hole footprints, as well as origin and bottom-side coordinate options. Agree these choices with the receiving manufacturer and document them. A DNP entry in a placement file is only interpretable in the context of that agreement. See the [KiCad 9 placement-file documentation](https://docs.kicad.org/9.0/en/pcbnew/pcbnew.html#_component_placement_files) for that version's options.

### 3. Geometry and assembly instructions

Open the manufacturing outputs in the receiving workflow or a suitable independent viewer. Check layer identification, outline, drilling and their alignment. Compare units, origin, side conventions and component orientation against the assembly drawing, including a representative bottom-side and polarized component when present.

Record viewer/tool version, file identity, view or layer, and the result. A screenshot should identify the location and explain what it demonstrates. If a check produces a DFx finding, attach the actual analysis image and source evidence to that finding.

This article's illustration explains the process. It is not a Gerber inspection image or evidence that any PCB has passed DFx.

### 4. Instructions needed at the next operation

Confirm applicable assembly notes, approved substitutions, firmware identity, programming settings, test limits, fixture revision, labeling and packaging. For each item, distinguish **available**, **reviewed**, and **approved for the stated scope**.

An available test procedure does not prove that a build passed testing. Similarly, package approval for a pilot build does not establish production performance, regulatory compliance or shipment acceptance.

## Worked Example: Resolving a Population Conflict

The following records are deliberately fictional and abbreviated. They demonstrate document reconciliation, not a validated design or a real factory instruction.

**Requested build:** DEMO-CTRL, variant BASE, one board per BOM quantity basis.

| Reference | BOM Rev 07 | Placement export Rev C | Assembly drawing Rev B |
|---|---|---|---|
| R10 | Fit; 1 per board | R10 present, top | Fit R10 |
| R11 | Fit; 1 per board | R11 present, top | Fit R11 |
| R12 | DNP; 0 fitted | R12 present; no population flag | Fit R12 |
| J1 | Fit; manual assembly | Omitted by declared export rule | Fit manually |

For this example, the agreed placement-file convention is **fitted machine-placed components only**. The expected reference set is `{R10, R11}`. The received set is `{R10, R11, R12}`. J1 is an explained manual-assembly exclusion; R12 is an unresolved conflict across the three instructions.

**Decision at this point: HOLD assembly release.** The reviewer cannot select the BOM as authoritative solely because its revision number is higher.

To resolve it, the design owner must confirm the intended BASE population. If that confirmation says R12 is DNP, update the assembly drawing, regenerate the applicable placement output, and repeat the comparison. Record the new file identities and evidence. If the intended population is different, correct the BOM and any other affected documents instead.

The example stops at the unresolved decision. It includes no invented approval, completed corrective action or production result.

## Record a Decision That Has a Clear Scope

Use decision names that match the organization's actual authority rules. The following are suggested working definitions:

| Decision | Meaning in this proposed workflow |
|---|---|
| PENDING | Review or decision is incomplete; no release authorization is recorded |
| HOLD | A blocker prevents the named activity; record the blocker, owner and required closure evidence |
| GO WITH CONDITIONS | An authorized person permits a precisely limited activity under recorded conditions, where the organization's policy allows this |
| GO | Required checks for the named scope have acceptable evidence and the authorized release decision is recorded |

For a conditional release, name the allowed activity, applicable quantity/lot, remaining restriction, owner and closure trigger. Avoid a vague statement such as "OK to proceed; documents later." An unresolved population conflict remains a blocker for the affected assembly activity.

A small NPI team can maintain the package index and collect specialist evidence without creating a new committee. Design, manufacturing, quality and customer decisions still go to the people who hold those responsibilities under the existing process. One coordinator does not automatically acquire all approval authority.

## Issue the Package and Preserve the Record

Once the decision is recorded, issue the exact reviewed files and manifest. Record the recipient, issue date and package identity in a transmittal. Ask the receiving party to confirm that the package is readable and that the intended configuration and scope are understood.

When a file changes, assess the affected checks, create a new package revision, and identify what it supersedes. Preserve the previous issued record. Do not replace a released file silently under the same package identity.

## Download the Working Templates

- [Release checklist and decision record — Markdown]({{ "/assets/downloads/engineering-release-package/release-checklist.md" | relative_url }})
- [Release manifest — Markdown]({{ "/assets/downloads/engineering-release-package/release-manifest.md" | relative_url }})
- [Release manifest — JSON]({{ "/assets/downloads/engineering-release-package/release-manifest.json" | relative_url }})
- [Template instructions and field definitions]({{ "/assets/downloads/engineering-release-package/template-guide.md" | relative_url }})

Start with the checklist and the Markdown manifest. The JSON version is an alternative for teams that want structured records; maintaining both is optional. When using both, designate one as authoritative and regenerate or reconcile the other before issue.

These blank templates contain no customer data or release authorization. Fill them with actual package evidence and use the approval rules applicable to the build.
