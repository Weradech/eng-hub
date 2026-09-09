# Worked example: DEMO-ECR-001

Fictional training case. No real manufacturer part, customer design, test result,
stock quantity or approval is represented. Values below are calculated from the
explicit assumptions. Formal decision: PENDING. Approved effectivity: not assigned.

## Request

- Product: DEMO-SENSE; variant: BASE (fictional identifiers).
- Proposed change: upper resistor R1 from 10 kΩ ±1% to 10 kΩ ±5%.
- R2, the lower resistor to ground, remains 10 kΩ ±1%.
- Package description for both R1 options: 0603; detailed compatibility unverified.
- Reason in the scenario: candidate availability. No actual availability is claimed.
- Requested scope: evaluation of a possible continuing substitution.
- Proposed production effectivity: not assigned.

## Model and exercise requirement

Vin is an ideal fixed 3.300 V. The output is unloaded. Only initial resistor
tolerance is modeled; tolerance extremes are independent. Nominal output is 1.650 V.
Temperature, aging, input-voltage variation, input loading and measurement/ADC
errors are excluded. Other component characteristics have not been evaluated.

Invented exercise requirement for this screen: 1.650 V ±2% = 1.6170–1.6830 V.
This requirement is not taken from a customer specification or manufacturer datasheet.

```text
Vout = Vin × R2 / (R1 + R2)
Vout_min = Vin × R2_min / (R1_max + R2_min)
Vout_max = Vin × R2_max / (R1_min + R2_max)

Original:
minimum = 3.300 × 9900 / (10100 + 9900) = 1.633500 V
maximum = 3.300 × 10100 / (9900 + 10100) = 1.666500 V

Candidate:
minimum = 3.300 × 9900 / (10500 + 9900) = 1.601470588… V
maximum = 3.300 × 10100 / (9500 + 10100) = 1.700510204… V
relative errors = (Vout / 1.650 - 1) × 100%
               = -2.941176…% and +3.061224…%
```

## Assessment

The original interval is within the exercise band for the modeled tolerance only.
The candidate interval extends outside it, so the candidate cannot guarantee that
requirement over its stated tolerance range. No measured failure is claimed.

Assessment recommendation: HOLD the candidate for production substitution.
Formal decision remains PENDING, with no approver or approval date.
The current exercise requirement is unchanged; it must not be widened merely to
accept the candidate. Approval of a real requirement change needs separate authority.

## Open work

| Area | Evidence still needed | State |
|---|---|---|
| Actual part compatibility | Real datasheets, package details, electrical/thermal ratings and provenance | NOT REVIEWED |
| Full sensing behavior | Loading, input range, temperature, aging and downstream measurement/threshold error analysis | NOT REVIEWED |
| Firmware / calibration | Impact on thresholds or scaling; evidence for any proposed compensation | NOT REVIEWED |
| Test | Requirement-linked method, criteria, configuration and sample rationale; actual results if executed | NOT REVIEWED |
| Stock / WIP / delivered units | Actual affected population, traceability and authorized disposition | NOT REVIEWED |
| Customer / compliance | Applicable obligations and decision evidence or N/A rationale | NOT REVIEWED |
| Implementation | Valid candidate, authorization, effectivity and controlled package update | NOT AUTHORIZED |

No new release package, rework, material disposal or production substitution is
authorized by this example. All real decisions must be made against actual inputs.

## Supporting reference

Texas Instruments, SLVA450B, discusses resistor divider ratio, leakage/loading and
accuracy considerations. It does not provide this exercise's inputs or results:
https://www.ti.com/lit/an/slva450b/slva450b.pdf
