---
title: "Screen Fixture Probe Clearance in 3D"
date: 2026-11-27 18:00:00 +0700
categories: [NPI, DFM]
tags: [dfx, pcba, fixture, test, clearance]
description: "Align board and fixture data, screen probe paths against fitted geometry, and record what remains to verify before fixture release."
---

> **TL;DR** — Match the board and assembly revisions, register the models to common datums and check the space occupied by each probe as it moves. Record close calls with a section view and the model assumptions.

A test pad can look accessible in plan view while a neighboring body intrudes into the probe barrel or fixture plate. Check each contact from its approach side with the board and fixture in the same coordinate system.

1. **Register the inputs.** Bring the test-point coordinates, fitted assembly variant and mechanical model into the same frame. Check units, origin and top/bottom orientation. Verify alignment against several locating features and record any residual offset. Mark the result provisional if revisions or fitted geometry cannot be confirmed.
2. **Screen the approach.** Check the probe tip, shaft and barrel wherever they enter the assembly space, including the travel from open to engaged. Check the plate and any relief pockets against component protrusions. Use the selected probe geometry and stroke when available, and record which assumptions still need confirmation.
3. **Resolve close calls.** Bounding boxes are useful for a first pass. If a box overlaps the probe envelope, inspect a section through the actual solids or use a finer mesh to account for chamfers and gaps. Compare the measured clearance with the allowance for board and fixture tolerances. Keep missing dimensions or uncertain geometry open.
4. **Record the disposition.** For each close or blocked point, capture the contact location and side, the model revisions, the minimum clearance or interference and a marked view from the analysis. Record the next action, such as checking a different probe or reviewing an alternate contact location.

Use the screen to prioritize fixture design work. Before release, verify the selected probe, travel and tolerance stack against the detailed fixture design, then confirm fit and contact on a representative assembly.

## Related Posts

- [A DFx Finding That Someone Else Can Verify]({% post_url 2026-09-29-dfx-finding-evidence %})
- [Engineering Release Package: Are These Files Ready to Build?]({% post_url 2026-09-09-engineering-release-package %})
