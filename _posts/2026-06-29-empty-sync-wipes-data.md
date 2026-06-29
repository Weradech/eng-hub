---
title: "The Empty Sync That Wiped the Dashboard: Carry-Forward Guards"
date: 2026-06-29 17:30:00 +0700
categories: [System, Data-Integrity]
tags: [data-integrity, sync, reliability, snapshot, etl, idempotency]
---

> **TL;DR** — A scheduled sync that overwrites a snapshot is one failed fetch away from erasing it. When the upstream returns zero rows because a token expired — not because the truth is zero — a naive sync writes "empty" over good data and the dashboard goes blank. The fix is to distinguish *empty-because-true* from *empty-because-failed*, carry forward the last good snapshot, and never overwrite in place.

---

## The morning the dashboard was blank

A revenue dashboard reads from a `revenue_snapshot` table. A scheduled job pulls figures from an upstream service every few hours and refreshes the snapshot. One morning the page rendered with everything at zero. No error banner, no alert — just a clean, empty, *confident-looking* dashboard.

The data wasn't lost. The previous snapshot was still sitting in the database one row back. What happened was simpler and worse:

```
sync job runs → upstream auth token had been rotated overnight
upstream returns 200 OK with an empty list (not a 401!)
job: "got the data, it's empty" → overwrites snapshot with 0 rows
dashboard reads the fresh snapshot → renders zero
```

The job did exactly what it was told. The bug was in what it was told to believe: that **an empty response means the real value is empty.**

---

## Empty-because-true vs empty-because-failed

Every sync has to answer one question about a zero-row payload: *is this the truth, or a failure that happens to look like the truth?*

| Situation | Rows returned | Correct interpretation |
|-----------|---------------|------------------------|
| Genuinely no data upstream | 0 | Truth is 0 — write it |
| Token expired, service returns empty 200 | 0 | Fetch failed — **do not** write |
| Network flap mid-page | partial | Fetch failed — **do not** write |
| Upstream had a bad deploy, returns `[]` | 0 | Fetch failed — **do not** write |

A naive job collapses all four rows into "I received a response, so I'll trust it." Three of the four then destroy data. The crucial realisation is that **most empty responses in a healthy system are failures, not facts** — a live revenue feed, an inventory list, a customer table rarely drops to genuinely zero, so "suddenly empty" should be treated as suspicious by default.

> A `200 OK` with an empty body is not the same as a confirmed-empty truth. Many APIs return an empty list on an expired session, a throttle, or a half-broken backend instead of a clean error code. Treat sudden emptiness as a failure signal until proven otherwise. (See [Network Errors Are Not Data]({% post_url 2026-06-13-it-said-connection-refused %}).)
{: .prompt-warning }

---

## Carry-forward: the last good snapshot survives

The defensive pattern is to refuse the overwrite unless the new payload clears a sanity bar, and otherwise keep serving the last known-good snapshot.

```
fetch upstream
if fetch failed (non-200, timeout, partial):
    keep last good snapshot, log a warning, raise an alert
    DO NOT write

if payload is empty AND last snapshot was non-empty:
    treat as suspicious → keep last good snapshot, alert
    DO NOT overwrite blindly

if payload passes the sanity floor:
    write a NEW snapshot row (append, don't overwrite in place)
```

Two design choices make this robust:

**Append, don't overwrite.** Write each refresh as a new immutable `revenue_snapshot` row with a timestamp, and let the dashboard read the latest. The previous snapshot is still there, so "carry forward" is just "don't write a bad new row" — the good one is already the latest valid one. This also gives you a free history to debug from. (When the blank-dashboard incident hit, the real numbers were recoverable precisely because the prior row still existed.)

**A sanity floor, not just a non-empty check.** "At least one row" is a weak guard — a half-broken upstream might return one row out of thousands. A better floor is relative: reject a payload that drops more than, say, 90% below the last good snapshot's size or total. A genuine cliff is rare; a sudden cliff is almost always a fetch problem.

---

## The guard is only as strong as its trigger

A carry-forward guard that nobody notices firing is its own trap. If the sync silently keeps serving yesterday's snapshot for a week because the token is still broken, the dashboard is *stale* instead of *blank* — quieter, but still wrong, and now it looks healthy.

So the guard has two halves:

1. **Refuse the bad write** (protects the data).
2. **Alert loudly that you refused** (protects against silent staleness).

Skip the second half and you've traded a visible failure for an invisible one. A blank dashboard at least tells you something broke; a confidently-stale one doesn't.

> A defensive guard that suppresses a symptom without surfacing the cause converts a loud failure into a silent one. Always pair "don't write the bad value" with "tell someone you didn't." A green dashboard serving week-old numbers is not a fixed bug.
{: .prompt-tip }

---

## The portable checklist

- **An empty response is a hypothesis, not a fact.** Confirm the fetch succeeded before you believe its emptiness.
- **Append snapshots; don't overwrite in place.** The last good value should survive any single bad run for free.
- **Use a relative sanity floor**, not just non-empty — catch partial payloads, not only fully-empty ones.
- **Alert when the guard fires.** Refusing a bad write and saying nothing turns a blank dashboard into a stale one.
- **Rotate secrets through the job, not around it.** This whole class of bug often starts with a token rotated in one place and not the other.

---

## Related Posts

- [It Said "Connection Refused"]({% post_url 2026-06-13-it-said-connection-refused %}) — a network error is not a data value
- [Is the Number Wrong, or Is the Stock Gone?]({% post_url 2026-06-13-reconciliation-without-false-alarms %}) — never act on a partial read
- [Cross-Path Idempotency]({% post_url 2026-06-13-cross-path-idempotency %}) — writing the same snapshot twice should be safe
