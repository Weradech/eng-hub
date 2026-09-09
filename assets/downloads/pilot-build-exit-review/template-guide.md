# Pilot build review guide

Use `pilot-build-review.md` for the review and `open-issue-log.md` for actions.
Both are blank and carry no release authorization. `demo-lot.json` is populated
synthetic training data; do not treat it as a factory record or import it into live systems.

## Using the records

1. Name the next activity for which a decision is requested.
2. Establish the actual configuration, unit population, process boundary and cut-off.
3. Reconcile unique units and preserve first-attempt, rework and retest histories.
4. Review requirements and execution evidence. Record unknowns and justified N/A items.
5. Separate unit recovery from issue closure; assign actual accepted owners.
6. Record the real authority, scoped decision and restrictions.
7. Route changes through the applicable change process and retain follow-up evidence.

Use one authoritative source per fact and link to it. Avoid maintaining duplicate
result tables in multiple documents when one unit-history source will suffice.

## Demo dataset conventions

- `synthetic`: true; all identities, events and issue observations are invented.
- `lot_id`, `product`, `variant`, `configuration`: fictional labels, not released documents.
- `checkpoint`: scope of the metric. Results apply to this checkpoint only.
- `snapshot_label`: a named exercise cut-off; it is not a real production timestamp.
- `planned_units` and `units_entered`: both 20 in this exercise.
- `units`: one object per physical unit, with a unique `unit_id`.
- `attempt_results`: ordered synthetic PASS/FAIL results at the defined checkpoint.
- `rework_performed`: whether physical unit rework occurred between the recorded attempts.
- `issue_ids`: links to synthetic observations in the `issues` array.
- `issues.proposed_owner_function`: suggested responsibility, not an assignment to a person.
- `assessment_recommendation`: editorial assessment, separate from the formal decision.
- `formal_decision`, `approved_next_scope`, `decided_by`: remain PENDING or null.

The demo has no pending first attempts, scrap or transfers. Every first-attempt PASS
has a single attempt and no rework. Every recovered unit has a first FAIL and later PASS.
The two remaining failed units each have one FAIL and are still held at the cut-off.
The demonstration model is intentionally narrow; real data may need more event types.

## Recalculation

Count unique units, not test attempts:

```text
N = number of distinct unit IDs = 20
first-pass units = one PASS attempt, no rework = 15
passed after rework = final PASS and rework_performed = true = 2
passed after retest without rework = initial FAIL, final PASS, no rework = 1
still failed = final FAIL = 2

15 + 2 + 1 + 2 = 20
checkpoint FPY = 15 / 20 = 75%
current passing proportion = 18 / 20 = 90%
reworked units = 2 / 20 = 10%
still failed / held = 2 / 20 = 10%
distinct retested units = 3 (includes the 2 reworked units)
total attempts = sum of attempt_results lengths = 23
```

The dataset's summary is derived from its unit records and can be recalculated.
Do not add overlapping rework/retest counts to the mutually exclusive outcome totals.
Current passing proportion is a snapshot, not a first-pass result, reliability
estimate or shipment release. No acceptance target is prescribed by this example.

For real datasets, decide how pending, transferred, scrapped and later-failing units
are handled before calculating metrics. Preserve all history and disclose the rule.

## Source for terminology

ASQ Quality Glossary, First pass yield:
https://asq.org/quality-resources/quality-glossary

The synthetic population, proposed review workflow and numerical example are
created for this article; ASQ does not provide or endorse those results.
