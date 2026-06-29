---
title: "SOP Hierarchy: When to Write a Procedure vs. a Work Instruction"
date: 2026-06-29 10:30:00 +0700
categories: [Process, ISO]
tags: [sop, iso, procedure, work-instruction, quality, manufacturing]
---

> **TL;DR** — Writing the wrong document type wastes everyone's time. Procedure (P) = multi-person, multi-department, outcome-focused. Work Instruction (WI) = one person, one task, step-by-step. The boundary is usually 1 person vs. N persons doing handoffs. A 30-page Procedure that operators can't follow on the floor is worse than no document at all.

---

![SOP hierarchy: a Procedure is for multiple people across departments and is outcome-focused; a Work Instruction is for one person doing one task, step by step. The boundary is one person versus several people doing handoffs]({{ "/assets/img/2026-06-29/sop-hierarchy.svg" | relative_url }})
_One person, one task → Work Instruction; several people with handoffs → Procedure._

## The ISO Document Hierarchy

ISO 9001 doesn't dictate a specific document structure, but the industry-standard interpretation produces a four-level hierarchy:

```
Level 1 │ Quality Manual / Policy
        │ "What we commit to and why"
        │ Audience: management, customers, auditors
        ▼
Level 2 │ Procedures (P)
        │ "How we manage cross-functional processes"
        │ Audience: department heads, process owners
        ▼
Level 3 │ Work Instructions (WI)
        │ "How one person does one task"
        │ Audience: operators, technicians on the floor
        ▼
Level 4 │ Forms / Records (FM)
        │ "What we captured and when"
        │ Audience: whoever needs evidence
```

Each level down is more specific, written by fewer people, and followed by more people. The Quality Manual is written once by the QMR; Work Instructions are written dozens of times by process owners and used daily by every operator.

The hierarchy matters because **audit findings cite the level, not just the document**. A nonconformance against a Procedure is a systemic gap; against a WI, it's a training or execution gap. They have different corrective action paths.

---

## The Decision Rule

The question to ask: **How many departments or role types are involved in this process from start to finish?**

| Situation | Document Type |
|-----------|--------------|
| Task involves handoffs between 2+ departments | Procedure (P) |
| Task is done by one person, start to finish | Work Instruction (WI) |
| Task is purely data capture, no decision required | Form / Record (FM) |
| Policy statement, no steps | Quality Manual entry |

**Common mistake:** Writing a Procedure when the process is actually a sequence of WIs stitched together. A 25-page Procedure that covers GR → IQC → storage → release is trying to serve three different audiences at once — the receiving dock, the QA lab, and the stores team. Each of those groups needs their own WI. The Procedure governs the handoff logic between them, not the step-by-step execution.

**Reverse mistake:** Writing a WI for a cross-departmental process because "the steps are clear." Steps can be clear and the document can still be the wrong type. If QA fails an IQC item and needs to notify Engineering and Procurement simultaneously, that decision logic belongs in a Procedure (responsibilities, escalation path), not a WI (do this, then this, then this).

---

## The MR Role: Approver, Not Author

One of the most common bottlenecks in ISO documentation programs is conflating the Management Representative (MR) role with the Author role.

**MR = Approver.** The MR signs Procedures to confirm they align with the QMS framework, regulatory requirements, and company policy. The MR is not the subject matter expert for every process they approve.

**Process Owner = Author.** The process owner writes the Procedure for their domain — they know the work, the exceptions, and the failure modes. An MR writing Procedures for processes they don't own produces generic, unusable documents.

When both roles are assigned to the same person (common in small companies), the bottleneck is structural: every new or revised document requires that person to context-switch into both author mode and approver mode, sequentially. The result is a queue that drains slowly.

The fix: separate the roles by naming a Procedure Author for each function (Receiving = SC02, QA = QA01, Production = PM03) even if the MR is still the final approver. Authors draft and maintain; MR reviews and signs. Review cycle drops from weeks to days.

---

## Procedure Anatomy

A Procedure answers "what happens across the organisation when this process runs?" It does not answer "what does the operator do next?"

**Standard sections:**

1. **Purpose** — One sentence: what outcome this procedure ensures.
2. **Scope** — What's in and what's out. Explicit scope prevents scope creep in audits.
3. **Definitions** — Terms that mean something specific in this context (e.g., "Nonconforming Material" = any item failing IQC criteria, regardless of source).
4. **Responsibilities (RACI)** — Who is Responsible, Accountable, Consulted, Informed for each step. A swimlane flowchart replaces a wall of text here.
5. **Process flow** — The cross-functional flow. Swimlane diagram with decision points and handoff arrows. Not step-by-step operator instructions.
6. **Referenced documents** — List WIs, Forms, and external standards referenced. This is how the hierarchy links.

What a Procedure does **not** contain: screen-by-screen software steps, tool-specific instructions, or anything that will change every time the software is updated. Those go in WIs.

---

## Work Instruction Anatomy

A WI answers "what does this person do, step by step, to complete this task correctly?"

**Standard sections:**

1. **Purpose** — One sentence: what correct completion of this task achieves.
2. **Scope** — Which products, systems, or conditions this WI applies to.
3. **Materials / Tools** — Everything needed before starting. Operator should not have to leave the workstation to find something.
4. **Steps** — Numbered, imperative mood ("Open the GR module. Scan the barcode. Verify the PO number matches."). Each step is one action. No compound steps.
5. **Accept / Reject criteria** — What good looks like, what bad looks like. Include photos or diagrams where visual inspection is involved.
6. **Safety notes** — Hazards specific to this task, not generic safety policy (that's the Policy Manual).

WIs are living documents. When the software UI changes, when a new product is introduced, when an operator finds a better sequence — the WI changes. Version control and revision history are not optional; an auditor will ask to see them.

---

## Real Example: GR → IQC Split

The receiving-to-IQC flow at Syntech started as a single Procedure (EN-P-04). It covered everything from PO receipt through IQC disposition to storage release.

When the QA team was asked to follow EN-P-04 on the floor, the response was: "This has 12 pages. I can't read this while scanning boxes."

That feedback identified the document type mismatch. EN-P-04 was a Procedure trying to serve as a WI. The split:

- **EN-P-04 Rev10** (Procedure) — Cross-functional flow: Procurement triggers GR, Receiving executes WI-GR-01, QA executes WI-IQC-01, disposition decision escalation path, Stores release criteria. 4 pages. For supervisors and auditors.
- **EN-WI-GR-01** (Work Instruction) — Step-by-step: Receiving operator opens WMS → scans shipment barcode → verifies PO match → prints GR label → moves to IQC staging. 2 pages + photos. For the dock.
- **EN-WI-IQC-01** (Work Instruction) — Step-by-step: QA opens IQC module → locates GR record → conducts inspection per sampling plan → records result → disposition. 3 pages + accept/reject photos. For the QA lab.

The Procedure is the management document. The WIs are the floor documents. An auditor reads the Procedure; an operator follows the WI.

---

## Naming Convention and Revision Control

```
Procedures:        EN-P-XX    Rev00 (first issue) → Rev01 → Rev02...
Work Instructions: EN-WI-XX   Rev00 → Rev01 → ...
Forms / Records:   FM-XX      Rev00 → Rev01 → ...
```

**Never break the naming framework when merging an external draft.** When a consultant or contractor delivers a document named "GR_Receiving_Process_v3_FINAL.docx", it goes into the system as EN-WI-GR-01 Rev00 after review — the external name disappears. The system's naming is the contract; external names are just inputs.

Revision notes must explain *why* the document changed, not just *what* changed. "Updated step 4 to reflect new WMS scan screen" is useful. "Rev01 updates" is not.

Keep a revision history table at the back of every controlled document:

| Rev | Date | Author | Change Summary | Approved By |
|-----|------|--------|---------------|-------------|
| 00 | 2026-04-15 | SC02 | Initial issue | MR |
| 01 | 2026-06-10 | SC02 | Updated scan step for WMS v2.3 | MR |
