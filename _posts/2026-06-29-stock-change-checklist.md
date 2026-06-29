---
title: "The Stock-Change Checklist: Never Mutate Live Inventory Blind"
date: 2026-06-29 18:30:00 +0700
categories: [Process, Operations]
tags: [inventory, data-integrity, operations, runbook, wms, checklist]
---

> **TL;DR** — Every direct change to live stock should be wrapped in the same three steps: capture the *before* state, make the change through a **single** code path, then verify the *after* state matches the intended delta. The two ways inventory data gets quietly corrupted are mixing a raw `UPDATE` with the service layer, and wiping-and-reinserting movements instead of applying a delta. The checklist exists to make both impossible to do by accident.

---

## Why a checklist, for something this routine

Adjusting a quantity feels trivial — it's one number. That's exactly why it's dangerous. The number isn't the inventory; it's a *summary* of a ledger of movements. Change the summary without respecting the ledger and you get a value that looks right and reconciles to nothing. The damage is invisible until a [reconciliation report]({% post_url 2026-06-13-reconciliation-without-false-alarms %}) or an audit surfaces it weeks later, long after anyone remembers the change.

A checklist turns "I'll just fix this quantity" into a procedure with a verifiable result. It's a runbook, not bureaucracy.

---

## The three steps

### 1. Capture the pre-state

Before touching anything, record what's there: the on-hand quantity, and ideally the movements that produced it. This is your baseline — without it you can't prove afterward whether the change did what you intended or something extra.

```
pre = on_hand(sku, location)      # write it down / log it
```

If you can't read the pre-state, you can't safely write. Stop here and fix your read path first.

### 2. Change through a single path

Make the change through **one** mechanism — the service layer that owns stock, the one that writes a movement *and* updates the balance together. Never reach around it.

> Never mix the service layer with a direct database write for the same change. The service layer posts a movement and updates the balance as one unit; a raw `UPDATE qty = ...` changes the balance and leaves no movement. Do both for one fix and the ledger and the balance now disagree — permanently, and with no audit trail to reconstruct what happened.
{: .prompt-warning }

This is the single most common way inventory data rots: someone "fixes" a discrepancy with `UPDATE t_inventory SET qty = 100 ...`, the number looks correct, and every movement-based recompute from then on produces a different answer because the movements never reflected the change. The balance and its own ledger have been forked.

### 3. Verify the post-state delta

After the change, read the state again and assert the change is *exactly* what you intended — no more, no less:

```
post = on_hand(sku, location)
assert post - pre == intended_delta     # not just "post looks plausible"
```

Checking that the new number "looks about right" is not verification. Verification is confirming the **delta** equals what you meant to apply. A change that moved the number by the wrong amount — because a concurrent operation also touched it, or because the change hit more rows than expected — only shows up when you check the difference, not the absolute value.

---

## The counting trap: apply a delta, don't wipe-and-reinsert

A physical count is where this goes wrong most often. The instinct after counting is to *set* the quantity to the counted figure — frequently implemented as "delete the existing movements and insert a fresh one." That destroys history.

The correct shape is to **apply the difference as a new movement**, preserving everything before it:

```
counted   = 95
system    = 100
adjustment = counted - system = -5

# RIGHT: post one adjustment movement of -5; history intact, balance now 95
# WRONG: delete movements, insert "count = 95"; you just erased the ledger
```

> Preserve movements after a count. A count produces an *adjustment*, which is itself a movement — `-5`, not "the new truth is 95." Wiping the prior movements to force the number throws away the audit trail that lets you ever reconcile again.
{: .prompt-tip }

The same principle defeats the reconciliation false-alarm in reverse: if every adjustment is a real movement, a recompute *from the movements* always reproduces the balance, and "the number disagrees with its own history" can't happen.

---

## When the source of truth is elsewhere

Sometimes the correct value lives in another system — an ERP holding the financial book of record. The temptation is still a direct `UPDATE` to match it. Don't. An authoritative correction is still a *movement* in your system, posted through the service layer, with the other system's value as the intended target. The [authoritative-overwrite pattern]({% post_url 2026-06-14-authoritative-overwrite-pattern %}) is how you make one system match another *and* keep a trail of having done so — not by editing the balance in place. And per [co-master ownership]({% post_url 2026-06-29-wms-odoo-comaster %}), a direct DB edit is precisely the move that breaks the contract between systems.

---

## The checklist, to keep at the keyboard

- [ ] **Pre-state captured** — on-hand (and movements) recorded before any change.
- [ ] **One path** — change goes through the stock service; no raw `UPDATE` alongside it.
- [ ] **Movement preserved** — the change posts a movement; nothing is wiped-and-reinserted.
- [ ] **Delta verified** — `post − pre == intended_delta`, not "looks plausible."
- [ ] **Cross-system corrections are movements too** — match the ERP via an authoritative posting, not an in-place edit.

Five boxes. Every one of them is something that, skipped, produces a number that reconciles to nothing — and a debugging session weeks later that starts with "this stock card is impossible."

---

## Related Posts

- [Is the Number Wrong, or Is the Stock Gone?]({% post_url 2026-06-13-reconciliation-without-false-alarms %}) — what skipping these steps looks like at reconciliation time
- [The Authoritative-Overwrite Pattern]({% post_url 2026-06-14-authoritative-overwrite-pattern %}) — matching another system without losing the trail
- [Co-Master Architecture]({% post_url 2026-06-29-wms-odoo-comaster %}) — why a direct DB edit breaks the contract between systems
