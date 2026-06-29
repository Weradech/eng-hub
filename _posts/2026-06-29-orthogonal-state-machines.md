---
title: "When One Status Column Isn't Enough: Splitting Orthogonal State Machines"
date: 2026-06-29 18:00:00 +0700
categories: [System, Data-Modeling]
tags: [data-modeling, state-machine, crm, schema-design, postgres, sql]
---

> **TL;DR** — A single `status` column that tries to capture lifecycle, pipeline position, and workflow step at once becomes a tangle of invalid combinations. When you find yourself inventing values like `won_but_waiting_for_paperwork`, you have two or three independent state machines crammed into one field. Split them into orthogonal columns, derive a display mirror for backward compatibility, and validate each axis transitions on its own.

---

## The smell: a status enum that keeps growing

A CRM lead starts with a clean `status` column: `new`, `contacted`, `qualified`, `won`, `lost`. Tidy.

Then the sales team asks for a pipeline view, so you add `proposal_sent` and `negotiation`. Then operations needs to know whether a won deal has been handed off, so you add `won_pending_handoff` and `won_delivered`. Six months later the enum looks like this:

```
new, contacted, qualified, proposal_sent, negotiation,
won, won_pending_handoff, won_delivered, won_invoiced,
lost, lost_to_competitor, on_hold, on_hold_qualified, reopened, ...
```

Every new requirement multiplies the list, because the column is being asked to answer three unrelated questions at once:

1. **Is this lead alive?** (lifecycle)
2. **Where is it in the sales pipeline?** (commercial stage)
3. **What does the team need to *do* next?** (workflow step)

Those are three different state machines. Forcing them through one column means every combination needs its own value — and most combinations are nonsense (`lost_negotiation_pending_handoff`), so you either invent guard logic to forbid them or quietly allow invalid rows.

> If adding one new business rule forces you to add several new enum values, the column is modelling a *product* of states, not a single state. That product grows multiplicatively; a set of orthogonal columns grows additively.
{: .prompt-warning }

---

## The fix: one column per axis

Give each independent question its own column, each with its own small, stable vocabulary:

| Axis | Column | Values | Owns the question |
|------|--------|--------|-------------------|
| Lifecycle | `lead_status` | `active`, `won`, `lost`, `dormant` | Is this lead alive? |
| Commercial | `opportunity_stage` | `prospect`, `qualified`, `proposal`, `negotiation`, `closed` | Where in the pipeline? |
| Workflow | `workflow_stage` | `new`, `in_progress`, `handoff`, `done` | What's the next action? |

Now the impossible combinations simply cannot be written without explicitly choosing all three values, and each axis has four-or-five values that rarely change. A "won deal awaiting handoff" is just:

```
lead_status      = won
opportunity_stage = closed
workflow_stage    = handoff
```

No new enum value was needed. The state already exists as a point in a small 3-dimensional space.

---

## Keep the old column — as a derived mirror

The hard part of this split is never the schema. It's the dozens of reports, filters, and frontend badges that already read `status`. Rip the column out and you break all of them in one deploy.

Instead, keep `status` as a **derived mirror** — a column whose value is computed from the three real axes, never written to directly:

```sql
-- conceptually: status is a projection of the three axes
status := case
  when lead_status = 'lost'                    then 'lost'
  when lead_status = 'won'
       and workflow_stage = 'handoff'          then 'won_pending_handoff'
  when lead_status = 'won'                     then 'won'
  when opportunity_stage = 'negotiation'       then 'negotiation'
  when opportunity_stage = 'proposal'          then 'proposal_sent'
  ...
end
```

Old consumers keep reading `status` and see exactly what they saw before. New consumers read the three axes directly. You migrate readers across at your own pace, and when the last one is gone you drop the mirror.

> A derived mirror buys you a gradual migration. The rule that makes it safe: the mirror is **read-only**. The moment one code path writes to `status` directly, it can disagree with the three axes it's supposed to summarise, and you're back to two sources of truth. (See [Co-Master Architecture]({% post_url 2026-06-29-wms-odoo-comaster %}) — the same single-owner discipline, applied within one table.)
{: .prompt-tip }

---

## Backfilling without inventing data

To split a live table you have to map every existing `status` value onto a `(lead_status, opportunity_stage, workflow_stage)` triple. Two failure modes here:

- **Lossy mapping.** If `won_invoiced` and `won_delivered` both map to the same triple, you've thrown away a distinction someone relied on. Audit the *distinct* values first and confirm each maps to a unique point — or that the collapse is intentional.
- **Silent invalid rows.** After backfill, assert that every row lands on a *legal* combination. A row that ends up `lost` + `negotiation` is a backfill bug, not a real state.

The acceptance test is a single query that should return zero:

```sql
-- count rows whose axes form an impossible combination
select count(*) from leads
where (lead_status = 'lost'  and opportunity_stage <> 'closed')
   or (lead_status = 'won'   and opportunity_stage <> 'closed')
   or (lead_status = 'active' and opportunity_stage  = 'closed');
-- expected: 0
```

Backfilling every row and getting `violations = 0` on this query is what tells you the migration is real, not just that it ran without error.

---

## Testing: each axis moves on its own

The whole point of the split is that the axes are independent, so the tests should prove it:

- Advancing `opportunity_stage` from `proposal` to `negotiation` must **not** touch `lead_status` or `workflow_stage`.
- Marking `lead_status = lost` is allowed from **any** pipeline stage (you can lose a deal mid-negotiation).
- `workflow_stage = handoff` is only meaningful once `lead_status = won` — that's the one cross-axis rule worth enforcing explicitly, and now you can write it as a single guard instead of forbidding a dozen enum values.

If a test that moves one axis forces an assertion on another, either you found a genuine coupling worth documenting — or the split isn't clean yet.

---

## When *not* to split

Orthogonal columns are the right call when the axes genuinely vary independently. They are overkill when:

- The "axes" are actually one machine with a linear path (`draft → review → published` — that's one column, leave it alone).
- You have two values total and no growth pressure.

The trigger to split is the smell at the top of this post: **new business rules forcing multiplicative enum growth, and invalid combinations you have to guard against.** Until you feel that, a single column is simpler and simpler wins.

---

## Related Posts

- [Co-Master Architecture]({% post_url 2026-06-29-wms-odoo-comaster %}) — single-owner discipline across two systems; this is the same idea inside one table
- [Is the Number Wrong, or Is the Stock Gone?]({% post_url 2026-06-13-reconciliation-without-false-alarms %}) — a derived mirror is a cache, and caches go stale
- [Alembic: Treat Revisions as a Contract]({% post_url 2026-06-29-alembic-contract %}) — how to ship the schema change behind this split safely
