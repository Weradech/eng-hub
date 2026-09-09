# Engineering release manifest

Blank working template. Status: DRAFT. Decision: PENDING.
Empty fields and file rows require completion; they do not represent verified files.

## Configuration

| Field | Value |
|---|---|
| Product / assembly | |
| Variant | |
| Package ID | |
| Package revision | |
| Source baseline / export record | |
| Intended activity | |
| Order / lot / serial range / quantity | |
| Quantity basis / boards per panel | |
| Authoritative manifest (this file or another named record) | |
| Prepared by / date | |
| Supersedes / change record | |

## Deliverable applicability

Record REQUIRED or N/A with the requirement source and a reason for N/A.
Add scope-specific items. A required but absent item remains an open blocker.

| Deliverable role | Applicability | Requirement source / N/A reason | Associated file paths or open item |
|---|---|---|---|
| Fabrication data and drilling | | | |
| Stack-up / fabrication requirements | | | |
| BOM and population variant | | | |
| Placement data / manual assembly list | | | |
| Assembly drawing and notes | | | |
| Firmware / programming instructions | | | |
| Test procedure / limits / fixture identity | | | |
| Labeling / packaging / traceability | | | |
| Scope-specific evidence | | | |

## File inventory

Add one row per actual controlled file. Use package-relative paths. Different
document revisions are acceptable only when their compatibility is established.
For an archive row, link an inventory of the archive's contents in the evidence column.
Do not include this manifest in its own file-hash inventory.

| Relative path | Role | Document revision | SHA-256 of final bytes | Source baseline / export record | Review evidence / compatibility record |
|---|---|---|---|---|---|
| | | | | | |

## Placement conventions

- Units / origin:
- Top / bottom coordinate and rotation conventions:
- DNP inclusion rule:
- Through-hole / manual component handling:
- Fiducial / non-component handling:
- Receiving manufacturer agreement reference:

## Review and decision

- Checklist path / revision:
- Open item record:
- Decision: PENDING
- Authorized activity / effectivity:
- Decision authority / role:
- Decided by / date:
- Decision evidence reference:
- Conditions, owners and closure triggers:

## Issue record

- Issued by / date:
- Recipient / transmittal reference:
- Final archive SHA-256 (external record only; leave empty if this manifest is inside that archive):
- Recipient acknowledgement reference / date:

A matching hash establishes byte identity only. A completed manifest is not a
substitute for technical review or the authorized release decision.
