# Pilot Build Exit Review

Blank working record. Formal decision: PENDING. No production or shipment
authorization is recorded. Use the existing authority rules for the named activity.

## Decision and population

- Review ID / revision / coordinator:
- Product / variant:
- Build lot / unique unit population reference:
- Review cut-off date and time:
- Next activity requested (trial / pilot / production lot / shipment):
- Requested scope, quantity, lot or serial range:
- Objectives / requirements / agreed exit criteria and source:
- Population size rationale and limitations:
- Checkpoint or process boundary used for metrics:

## Configuration actually built

| Item | Intended released identity / revision | Actual identity / revision | Affected units / evidence | Discrepancy / action ID |
|---|---|---|---|---|
| Release package / manifest | | | | |
| PCB / BOM / fitted-part variant | | | | |
| Firmware / configuration / calibration | | | | |
| Assembly instructions / deviations | | | | |
| Test method / limits / fixture / equipment | | | | |

## Unit reconciliation

Use unique physical unit IDs. Define mutually exclusive current categories.
Retain first-attempt and recovery history. Record transferred units and exclusions
with a reason; never remove unresolved units just to improve a metric.

- Planned units:
- Units entered the defined boundary:
- Units with a first-attempt result:
- Units pending an initial result:
- Units transferred outside the reviewed population / reason / destination:
- Authoritative unit-history record:

| Current category | Count | Unit IDs / evidence |
|---|---|---|
| Passed first attempt without rework/retest within the boundary | | |
| Currently passed after rework | | |
| Currently passed after retest without rework | | |
| Still failed / held | | |
| Pending initial result / in process | | |
| Recorded scrap, if applicable | | |
| Other defined category, if applicable | | |
| Reconciled total | | |

The count for every unit must appear in exactly one current category. If a unit
passes then later fails, preserve its history and classify its current state honestly.
Investigate contradictions before finalizing the review.

## Metrics and interpretation

| Metric | Numerator / denominator / rule | Result | Source / cut-off / limitation |
|---|---|---|---|
| First-pass yield at the stated boundary | | | |
| Current passing proportion | | | |
| Units reworked | | | |
| Units retested, including any reworked units | | | |
| Still failed / held | | | |
| Total test attempts (event count, not unit denominator) | | | |

Reworked and retested unit counts can overlap. Rates with incomplete processing
need an explicit provisional rule. Do not present current passing proportion as FPY.

## Readiness evidence

Status: NOT REVIEWED / SUPPORTED / BLOCKED / N/A WITH REASON.

| Area | Status | Evidence / limitation | Action ID / owner |
|---|---|---|---|
| Configuration and traceability | NOT REVIEWED | | |
| Applicable product requirements / acceptance evidence | NOT REVIEWED | | |
| Assembly instructions / controlled rework | NOT REVIEWED | | |
| Test method / fixture / equipment / inconsistent results | NOT REVIEWED | | |
| Materials / kits / WIP disposition | NOT REVIEWED | | |
| Operator preparation / handoff | NOT REVIEWED | | |
| Required execution time / capacity evidence | NOT REVIEWED | | |
| Customer and applicable compliance conditions | NOT REVIEWED | | |
| Open-issue closure / effectiveness evidence | NOT REVIEWED | | |

## Recommendation and formal decision

- Open Issue Log path / revision:
- Engineering assessment recommendation and rationale:
- Formal decision: PENDING
- Authorized next activity / quantity / lot / serial scope:
- Decision authority / role:
- Decided by / date / evidence reference:
- Conditions / restrictions / action owners / closure triggers:
- Applicable customer decision or justified N/A:
- Shipment decision reference, if shipment is within this review's scope:

Suggested decisions: PENDING, HOLD, GO WITH CONDITIONS, GO. Apply existing
authority rules. A recommendation or a passing unit does not itself authorize release.

## Follow-up

- Required change records / updated release package:
- Next review trigger / responsible coordinator:
- Instructions issued and acknowledgement evidence:
- Closure evidence for required follow-up:
- Remaining limitations:

Optional method evaluation (actual observations only): preparation time, missing
evidence found and repeated open issues at the next build. No target or saving is assumed.
