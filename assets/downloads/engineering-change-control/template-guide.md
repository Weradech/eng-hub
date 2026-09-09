# Engineering change record guide

Use the Markdown assessment as one working record from request through closure.
The JSON is optional structured data, not an automated approval tool or JSON Schema.
If both representations are maintained, identify the authoritative one and reconcile
the other before issue. Keep decisions and measured results empty until evidenced.

## Workflow

1. Identify the current released configuration and exact proposed change.
2. Separate the request for evaluation from any authorization for production use.
3. Review affected interfaces and record evidence or justified N/A conclusions.
4. Define verification conditions and criteria before testing; retain result provenance.
5. Obtain the applicable formal decision and material/build disposition decisions.
6. Define approved effectivity; update the affected controlled records and issue a package.
7. Verify actual implementation on the first affected build and close required follow-up.

Requested effectivity is a proposal. Approved effectivity is a separate decision field.
An engineering recommendation is also separate from the formal decision.
An unused array indicates that nothing has been recorded, not that a review passed.
Unknown quantities and costs should remain null or explicitly unknown, never assumed zero.

## JSON field conventions

- `template_version`: version of the template structure, separate from record revision.
- `record_status`: suggested lifecycle values are DRAFT, IN REVIEW, IMPLEMENTING, CLOSED.
- `request.current_part` and `request.candidate_part`: identify manufacturer, exact part
  number, controlled datasheet revision/source and package description.
- `request.evaluation_authorization`: actual authority, scope, evidence and restrictions
  for a sample purchase or controlled engineering trial, if relevant.
- `request.proposed_effectivity` and `decision.approved_effectivity`: product/variant,
  applicable order, lot, serial range, quantity/unit and implementation boundary.
- Dates: ISO 8601; include timezone offsets for timestamps.
- Evidence: controlled record or package-relative path, plus page, location or item ID.

Populate arrays using the following fields; retain null until a value is known:

| Array | Fields for each entry |
|---|---|
| `impact_assessment` | `id`, `area`, `status`, `impact_or_rationale`, `evidence`, `owner`, `reviewed_at` |
| `verification` | `requirement_and_source`, `method`, `conditions`, `sample_rationale`, `acceptance_criterion`, `tested_configuration`, `analysis_result`, `analysis_evidence`, `measured_result`, `measurement_evidence`, `reviewer`, `reviewed_at` |
| `open_actions` | `id`, `issue`, `affected_activity`, `required_evidence`, `owner`, `due_date_or_trigger`, `closure_evidence`, `closure_reviewer`, `closed_at` |
| `material_disposition` | `population`, `location`, `actual_configuration`, `quantity`, `unit`, `proposed_disposition`, `approved_disposition`, `authority`, `decided_by`, `decided_at`, `decision_evidence`, `implementation_evidence` |
| `decision.conditions` | `restriction`, `permitted_scope`, `owner`, `closure_trigger`, `evidence` |
| `implementation.controlled_item_changes` | `item`, `current_revision`, `new_revision_or_carry_forward`, `change_or_rationale`, `implementer`, `implemented_at`, `evidence` |
| `implementation.recipient_acknowledgements` | `recipient_role`, `instruction_reference`, `acknowledgement_evidence`, `acknowledged_at` |
| `implementation.remaining_follow_up` | `action`, `owner`, `due_date_or_trigger`, `evidence` |

Suggested impact statuses: NOT REVIEWED, AFFECTED, UNAFFECTED WITH EVIDENCE,
N/A WITH REASON. Provide rationale/evidence for each conclusion.

`decision.status` defaults to PENDING. Suggested alternatives are HOLD, REJECTED,
APPROVED FOR LIMITED SCOPE, APPROVED; adapt these to existing authority rules.
Record the person, role, date, evidence, scope and conditions of the actual decision.

`closure.status` defaults to PENDING; use CLOSED only when required implementation
and follow-up evidence supports closure by the appropriate authority.

## Example boundary

`worked-example.md` is a fictional calculation exercise. Its HOLD recommendation
is an engineering assessment of the stated model, not a completed approval record.
Do not copy its values into a real change without actual requirements and part data.

For each real change, preserve the previous configuration and retain the issued
new package with the effectivity and supersession record. Returning to a previous
configuration also requires a valid scoped decision.
