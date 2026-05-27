---
title: "SOP Revamp Playbook — 13 Principles for SOPs That Actually Get Used"
date: 2026-05-27 14:00:00 +0700
categories: [ISO, Documentation]
tags: [sop, iso9001, work-instruction, process-documentation, quality]
---

> **TL;DR** — A good SOP is not the longest one. It's the one a new employee can follow immediately. These 13 principles come from an actual EN department document overhaul.

---

## The Problem with Legacy SOPs

Most SOPs in Thai SMEs share the same issues:
- Written once, never updated
- Too long → nobody reads them
- Vague language ("should", "approximately", "as needed")
- No clear Process Owner → no accountability
- Written for auditors, not users

---

## ISO Document Hierarchy

```
Tier 1 — Quality Manual
    ↓
Tier 2 — Procedure (P) — "What, Why, Who"
    ↓
Tier 3 — Work Instruction (WI) — "How, step-by-step"
    ↓
Tier 4 — Form / Record (FM) — capture results
```

**Common mistake:** Writing a Procedure with the detail level of a WI → duplication, hard to maintain.

---

## The 13 Principles

### 1. Decide the Document Tier Before Writing
Determine whether you're writing a P or WI first. Never mix them.

### 2. Header Must Have 6 Fields
```
Doc No: EN-P-001          Rev: 02
Title:  NPI Gate Review Process
Effective Date: 2026-05-01
Process Owner: NPI Manager
Approved by: MR (Management Representative)
```
**MR = Approver always** — not the Department Head.

### 3. Process Owner ≠ Author
The Process Owner is the person who:
- Is accountable for the process output
- Must update the SOP when the process changes
- Gets questioned during audits

### 4. Assign Ownership by Business Function

| Function | Process Owner |
|----------|--------------|
| NPI Gate | NPI Manager |
| Drawing Release | R&D Lead |
| Production | Production Manager |
| IQC | QA Manager |

### 5. Use Action-Oriented Language

| ❌ Vague | ✅ Clear |
|---------|---------|
| Should inspect | Inspect [item X] using [tool Y] |
| Approximately | ± 0.5 mm |
| Sometimes | Every time [condition Z] is met |

### 6. Adapt Gate Checkpoints to Risk Level
Not every step needs the same checkpoint weight:
- High-risk step → mandatory checkpoint + sign-off
- Routine step → self-check is sufficient

### 7. Explain the "Why"
```
❌ "Verify BOM before issuing PO"
✅ "Verify BOM before issuing PO because suppliers do not accept
   claims for incorrect part numbers — resulting in cost increases
   and delivery delays"
```

### 8. Apply ISO Compliance Selectively
Include what ISO requires. Don't invent additional bureaucracy:
- ✅ Risk assessment required (ISO 9001:2015 clause 6.1)
- ❌ No need for approval signatures on every revision if e-approval is available

### 9. Cut the Bloat
Common low-value sections to trim:
- 3-paragraph "Objective" → one sentence
- Definitions that everyone already knows → remove
- Reference list that nobody opens → keep only what's actively used

### 10. Numbering Hygiene
```
✅ Use:   4.1 / 4.2 / 4.2.1
❌ Avoid: 4.1.1.1.1  (more than 3 levels deep)
❌ Avoid: Section A, Section B (mixed with numbers = confusion)
```

### 11. Procedures Can Be Intentionally Loose
A Procedure (P) can be broad — the detail belongs in the WI.
Don't try to cover every edge case at the P level.

### 12. Run an Auto-scan Before Every Release
- [ ] Is the document number unique?
- [ ] Is the revision correct?
- [ ] Has the Process Owner approved?
- [ ] Are all referenced documents still active?

### 13. Treat SOPs as Living Documents
Update when:
- A non-conformance occurs from that process
- The actual process changes in practice
- Audit feedback received
- A new employee follows it and gets confused

**Target: review every 12 months or when a trigger event occurs.**

---

## Work Instruction Template

```markdown
Doc No: [EN-WI-XXX]    Rev: [00]    Date: [YYYY-MM-DD]
Title: [WI Name]
Process Owner: [Name + Title]
Approved by: [MR]

## 1. Purpose
This WI describes how to [action] in order to [outcome].

## 2. Scope
Applies to [define scope].

## 3. Tools / Materials Required
- [List items]

## 4. Steps
1. [Action verb] + [object] + [acceptance criterion]
2. ✓ Checkpoint: [what to verify, how]
3. ...

## 5. Related Documents
- [Doc No]: [Title]

## 6. Record
Record results in [Form No]: [Form Name]
```

---

## Summary

A good SOP = **Short + Clear + Usable + Owned + Maintainable**

If an SOP exceeds 3 pages for a single task — split it into multiple WIs.
