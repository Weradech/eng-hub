---
title: "The Kaizen 5-Section Report: A Format That Gets Decisions Made"
date: 2026-06-29 10:20:00 +0700
categories: [Process, Reporting]
tags: [kaizen, reporting, lean, process, management, html]
---

> **TL;DR** — Most engineering reports describe problems but don't drive decisions. The 5-section Kaizen format forces a decision path: Pain → State → Solution → Metrics → PDCA. Each section has one job, and none of them is "show how much work I did."

---

![The Kaizen 5-section report as a decision path: Pain, State, Solution, Metrics, PDCA, each section with one job that forces a decision]({{ "/assets/img/2026-06-29/kaizen-report-pattern.svg" | relative_url }})
_Five sections, each with one job &#8212; and each one forces a decision the reader otherwise wouldn&#8217;t make._

## Why Most Engineering Reports Fail

The failure mode is consistent across industries: the report is written for the author, not the reader.

**Too much data, no signal.** A 12-page report with 8 tables of raw numbers requires the manager to do the analysis that the engineer should have done. If the conclusion isn't stated, most managers won't draw one — they'll ask for a meeting instead.

**No baseline.** "Failure rate improved" means nothing without a before number. "Failure rate dropped from 18% to 3%" is a decision. The baseline is the entire point.

**No recommended action.** Engineering reports often end with findings. Kaizen reports end with an owner, a date, and the next step. The difference is whether a decision happens in the meeting or after the meeting.

**Written after the fact.** A report written a week after the incident reconstructs from memory. Memory is wrong. Numbers drift. A report written from live data at the time of writing is accurate; one written from memory is a narrative.

---

## The 5 Sections

Each section has exactly one job. If a section is trying to do two jobs, split it.

### 1. Pain

**Job:** State what broke, in plain language, quantified.

Not: "There were issues with the WMS inventory sync process resulting in discrepancies."  
But: "WMS stock count was overstated by 1,623 items vs Odoo on June 12. Value: approx ฿2.1M in incorrect inventory position."

Pain must be:
- Quantified (number, unit, time)
- Stated in terms the decision-maker cares about (money, time, customer impact)
- Free of jargon — if the word only means something to engineers, remove it

One paragraph. Maximum.

### 2. State

**Job:** Show the current situation with live numbers.

Pull data fresh at the time of writing. Never use numbers from memory, a previous report, or a gut feeling. If you can't pull it live, say so and note when the data was last verified.

State answers: "What is actually happening right now?"

Not "as of last week" — as of the moment this report was written.

### 3. Solution

**Job:** State what was done or what is proposed. Be specific.

Not: "We will improve the sync process."  
But: "Deployed commit `fix/sync-idempotent` — GR write is now atomic with WMS ledger update. Rolled out 14:30 Jun 13. Root cause: non-idempotent partial write on network retry."

If the solution is a proposal (not yet done): state the option, the cost, and your recommendation. Don't present three equal options and let the manager choose blind — pick one and say why.

### 4. Metrics

**Job:** Before/after numbers. The evidence that the solution worked (or didn't).

This is the only section where charts belong. One chart, if it helps. Two charts is too many for a Kaizen report — this isn't a dashboard.

```javascript
// Chart.js pattern: before/after bar
new Chart(ctx, {
  type: 'bar',
  data: {
    labels: ['Before (Jun 12)', 'After (Jun 14)'],
    datasets: [{
      label: 'Inventory Discrepancy (items)',
      data: [1623, 0],
      backgroundColor: ['#ef4444', '#22c55e']
    }]
  },
  options: { plugins: { legend: { display: false } } }
});
```

If you can't show a before/after, you don't have a Metrics section — you have another State section. Go back and find the baseline.

### 5. PDCA

**Job:** Next steps with owner and date.

| Phase | Item | Owner | Date |
|-------|------|-------|------|
| Plan | Monitor sync error rate daily | SC02 | Ongoing |
| Do | Deploy reconciliation page `/verify` | Engineering | Jun 15 |
| Check | Verify 0 discrepancy in next cycle-count | SC03 | Jun 20 |
| Act | Add to standard runbook if stable | QA | Jun 27 |

No PDCA item without an owner. "Team" is not an owner.

---

## The Golden Rules

**Pull live numbers every time.**  
A report written from memory is a story. A report written from live data is evidence. Before writing State or Metrics, open the database, run the query, check the dashboard. It takes 10 minutes and it's the difference between a credible report and a narrative.

**ELI5 language in the exec section.**  
If you wouldn't say it to someone's parent who's never worked in manufacturing, rewrite it. "GR non-idempotent write with WMS ledger partial commit" → "When the network dropped during a receive operation, the system wrote half the data and thought it was done. Repeat scans doubled the count."

**One sentence conclusion per section.**  
End every section with a conclusion sentence — what the reader should take away. If you can't write that sentence, the section isn't done.

**Include a baseline.**  
"Improved" without a before number is marketing. "Dropped from 18% to 3%" is evidence. The baseline is the whole point of the report.

---

## HTML Template Pattern

For exec-level distribution, a self-contained HTML file beats a PDF:
- Opens on any device, no software needed
- Charts render live (Chart.js CDN)
- Single file, email as attachment or share via Cloudflare Pages
- Dark theme reads well on screens

The 8-section skeleton:

```
Cover          → Project name, date, author, one-line summary
Exec KPI       → 3–5 KPI cards (big number + label + delta)
Pain           → Section 1: what broke
State          → Section 2: current situation
Solution       → Section 3: what was done
Metrics        → Section 4: Chart.js before/after
PDCA           → Section 5: table with owner + date
Vision         → One paragraph: where this leads (optional)
```

Keep it under 1,200px wide. Use a dark background (`#0f1117`), card surfaces (`#1a1d27`), and accent colours only for deltas and alerts. No corporate clip art.

---

## What NOT to Include

**Internal ratings or scores after people's names.** If the report leaves the building, this is a legal and HR problem. Even internally, scores next to names change the dynamic of the meeting.

**Commit hashes.** Managers don't need them. Engineers can find them in git. If you must reference a deploy, say "deployed June 13 at 14:30" — that's enough to find it.

**AI squad or internal tool names.** Exec reports describe outcomes, not tools. "WALAI automated the sync reconciliation" → "The reconciliation now runs automatically every 15 minutes."

**Financial forecasts.** Weekly and incident reports should not contain revenue projections or cost forecasts unless explicitly requested. Forecasts belong in a separate planning document where assumptions can be interrogated properly.

**Everything that happened.** A Kaizen report is not a changelog. If it's not relevant to the Pain/Solution/Metrics loop, it goes in the engineering log, not the report.
