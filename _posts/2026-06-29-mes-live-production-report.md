---
title: "Building a Live Production Report from MES Traceability API"
date: 2026-06-29 11:10:00 +0700
categories: [System, MES]
tags: [mes, reporting, api, chart-js, production, traceability, html]
---

> **TL;DR** — A production report that pulls live data at generation time is worth 10× a static PowerPoint. One API call + one standalone HTML file = a shareable, accurate view of your manufacturing floor. No dashboard server, no login, no stale screenshots.

---

![A static deck goes obsolete on arrival; a standalone HTML report that fetches the MES traceability API at generation time stays accurate]({{ "/assets/img/2026-06-29/mes-live-production-report.svg" | relative_url }})
_Live data at generation time beats a static deck 10× — one API call bakes the floor's real numbers into a shareable HTML file._

## The Data Model

Every unit entering the PCBA test station gets a serial number stamped at test time. The traceability system records two events per unit:

**PCBA test event**: `serial`, `mac`, per-step results (`power`, `rails`, `lan`, `usb`, `ping`, `rs485`), overall `pcba_result` (PASS/FAIL), `pcba_at` timestamp.

**Box build event**: `box_barcode`, `box_result` (PASS/FAIL), `box_at` timestamp, `box_matched` (whether the PCBA serial matches what the box scan expected).

From these two events, every downstream metric derives: FPY, step failure rate, jig comparison, retest count, time-in-test, final disposition. The schema is minimal — two tables joined on `serial` — but it answers every question a production manager or quality engineer would ask.

The final status field rolls up the combination:

| PCBA | Box | Status |
|------|-----|--------|
| PASS | PASS + matched | `DONE` |
| PASS | not yet | `TESTED` |
| FAIL | — | `TEST_FAIL` |
| PASS | PASS + mismatch | `BOX_MISMATCH` |

---

## The API Surface

Three endpoints cover everything from exec KPI to unit-level genealogy:

**`GET /api/trace/summary`** — fast, lightweight, good for dashboards that auto-refresh:
```json
{
  "statuses": { "total": 510, "DONE": 398, "TESTED": 106, "TEST_FAIL": 3 },
  "pcba_test": { "total": 686, "pass": 525, "fail": 161 },
  "box_build": { "total": 430, "pass": 413, "fail": 17 }
}
```

**`GET /api/trace/analytics`** — full analytics payload, everything needed for a production report:
- `fpy`: first-pass yield (units that passed on the first test attempt)
- `steps[]`: per-step pass/fail counts across all test runs
- `jigs[]`: per-jig test count, pass count, average duration
- `daily[]`: per-day test volume and pass/fail counts
- `hourly[]`: per-hour breakdown (shift analysis)
- `stepTrend[]`: per-step fail count by day (for trending)
- `retest[]`: distribution of how many tests each unit needed
- `repeat[]`: units with the highest test counts (watch list)
- `duration`: avg/median/min/max test time in seconds

**`GET /api/trace/board?limit=N`** — unit-level table, one row per serial, all fields. Use for drill-down and export.

---

## What the Analytics Reveal That the Live UI Doesn't

The live shop-floor display shows current status. The analytics endpoint shows patterns — and patterns are where problems hide.

**Step failure correlation.** If `lan`, `ping`, `usb`, and `rs485` all fail at similar rates on the same day, they're not independent failures. Those four steps all depend on the firmware network stack. A correlated failure across all four points to a firmware image issue or a test fixture problem, not four separate component defects. If only `power` and `rails` were high, you'd look at hardware — power supply, voltage regulator. The correlation is the diagnosis.

**Jig comparison.** Two jigs running the same test program on the same product should show similar pass rates. A 72.9% pass rate on JIG_A vs 80.8% on JIG_B with the same batch of boards means the jigs have drifted — contact quality, cable wear, fixture alignment. Without the analytics split by jig, this is invisible.

**Retest distribution.** 438 units passing on the first attempt, 46 needing two attempts, and 3 units failing 8+ times in a row are three completely different problems. The 438 are process-stable. The 46 are marginal units that benefit from a retest cycle — worth investigating whether a process parameter (solder joint, connector seating) is causing them. The 3 are almost certainly hardware defects: scrap candidates, not retest candidates.

**Daily trend as a process change detector.** Pass rate dropping from 90.8% on Jun 27 to 52.8% on Jun 29 is not random variation. Something changed. It could be a new firmware build, a fixture swap, a batch of components from a different reel, or a process change on the line. The daily trend doesn't tell you which one — but it tells you exactly when to look, which narrows the investigation to a 24-hour window.

---

## Building the Report

The report structure maps one-to-one with the analytics payload:

```
KPI cards:     units.total, units.DONE, fpy.first_pass/fpy.units, box.pass/box.total
Status donut:  units.DONE + units.TESTED + units.TEST_FAIL + units.BOX_MISMATCH
Daily chart:   daily[].tests, daily[].pass, daily[].fail (grouped bar)
Step table:    steps[] sorted by fail rate descending, with inline bar chart
Jig table:     jigs[].pass/tests, jigs[].avg_s
Step trend:    stepTrend[] as multi-line chart, one line per step
Retest dist:   retest[] as bar chart
Watch list:    repeat[] filtered to ≥4 tests, flag still-FAIL units
```

The fetch and render pattern:

```javascript
const data = await fetch('/api/trace/analytics').then(r => r.json());
const fpy = (data.fpy.first_pass / data.fpy.units * 100).toFixed(1);

// KPI card
document.getElementById('fpy-value').textContent = fpy + '%';

// Chart.js daily bar
new Chart(document.getElementById('dailyChart'), {
  type: 'bar',
  data: {
    labels: data.daily.map(d => d.day),
    datasets: [
      { label: 'Pass', data: data.daily.map(d => d.pass), backgroundColor: '#22c55e' },
      { label: 'Fail', data: data.daily.map(d => d.fail), backgroundColor: '#ef4444' }
    ]
  }
});
```

The entire report is one standalone HTML file with Chart.js loaded from CDN. No build step. Open it in a browser, print to PDF, email as attachment, or drop it on Cloudflare Pages for a shareable link.

---

## The Alert Logic

Compute today's fail rate and compare it to the previous session's:

```javascript
const days = data.daily;
const today = days[days.length - 1];
const prev  = days[days.length - 2];

const todayRate = today.fail / today.tests;
const prevRate  = prev.fail / prev.tests;
const delta     = todayRate - prevRate;

if (delta > 0.20) {
  // render red alert bar at top of report
  showAlert(`Fail rate surged ${(delta*100).toFixed(1)}pp vs previous session`);
}
```

The Jun 29 case: 9.2% (Jun 27) → 47.2% (Jun 29) = +38pp. Alert fires. The step breakdown confirms LAN/PING/USB/RS485 all failed at ~44% while POWER and RAILS stayed normal. That pattern — connectivity steps failing together, power steps stable — points directly at firmware or fixture, not hardware. Engineering gets the right scope for their RCA before they open the defective boards.

---

## Standalone HTML Advantages

A dashboard server requires uptime, authentication, and maintenance. A standalone HTML file requires none of these:

- **Email as attachment**: works offline, no VPN required, no login
- **Print to PDF**: faithful rendering of charts, suitable for customer or management review
- **Cloudflare Pages**: drag-and-drop deploy for external stakeholder access, no server cost
- **Version history**: save a copy each shift. The report is a snapshot. Compare across shifts without a time-series database.
- **No auth surface**: the report contains no credentials and serves no live data. The data is baked in at generation time. Safe to share with external auditors or customers.

The tradeoff: the report is a point-in-time snapshot. It doesn't auto-refresh. For real-time monitoring, the live MES UI is the right surface. For reporting, communication, and analysis, the standalone HTML wins on simplicity and portability.

---

## What to Add Next

| Feature | Value | Complexity |
|---------|-------|------------|
| Date range picker | Filter analytics to a specific run or week | Medium |
| Project filter | Multiple product lines on the same jig | Low — add `?project=SGW` param |
| Shift breakdown | Hourly chart is already there; label by shift | Low |
| Auto-alert email | Send report when delta > threshold | Medium — needs a mailer |
| PDF auto-generation | Headless Chrome + cron at shift end | Medium |

The date range picker is the highest-value addition: right now the analytics are all-time, which means the step failure rates include early production ramp (higher fail rates) alongside stable production (lower rates). Filtering to the last 48 hours gives a much cleaner signal for process control.
