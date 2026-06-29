---
title: "A 403 on Your Own Automation Doesn't Mean Auth Is Healthy"
date: 2026-06-29 16:00:00 +0700
categories: [Infrastructure, DevOps]
tags: [devops, monitoring, auth, observability, rca, reliability]
---

> **TL;DR** — When a scheduled job starts returning 403, the comforting read is "auth is working — it's correctly rejecting a bad request." But a 403 against *your own automation* usually means the job's credential died, not that security is healthy. A liveness check that only asks "does the endpoint respond?" will show green while a background job has been silently failing hundreds of times.

---

![One 403 has two opposite meanings: a control working, or your own job's credential dead behind a green dashboard]({{ "/assets/img/2026-06-29/403-not-auth-healthy.svg" | relative_url }})
_Same status code, opposite meaning — identify whose credential failed before you call a 403 "expected."_

## The false comfort of a rejection

An automated planner ran on a schedule, calling an internal API to recompute and post a plan. At some point it started getting `403 Forbidden`. Someone glanced at it and reasoned:

> "403 means the auth layer is doing its job — it's rejecting something. The security side is fine; this is just my request being wrong."

Both halves of that sentence are technically true and together they're dangerously misleading. Yes, *something* was rejected. No, that does **not** mean the system is healthy. The job's JWT secret had been rotated out from under it. Every scheduled run since had failed — hundreds of silent execution errors — and the automation it was supposed to drive had quietly stopped happening. The plan wasn't being recomputed at all.

The 403 wasn't auth *succeeding*. It was auth *failing to authenticate a client that should have been authenticated* — a broken integration wearing the costume of a working security control.

---

## "The endpoint rejected me" ≠ "auth is healthy"

The trap is collapsing two different facts into one word:

| What you observe | What it could mean | What it does **not** prove |
|------------------|--------------------|-----------------------------|
| `403` from the API | Your token expired / was rotated | That auth is "working as intended" |
| `403` from the API | A genuine permission boundary held | That *your job* is fine |
| `200` from a health check | The service is up | That your authenticated job can actually call it |

A 403 tells you a request was refused. It says nothing about *whether the refusal is correct*. To know that, you have to separate the **source** of the 403:

- **The service's policy is doing its job** (a genuinely unauthorised caller was blocked) → healthy.
- **Your own client's credential is broken** (expired, rotated, misconfigured) → a broken integration, not a security win.

Treating the second as if it were the first is how a dead automation gets logged as "auth healthy, no action needed."

> Before you call a 403 "expected," identify *whose* credential failed. A rejection of an attacker is a control working. A rejection of your own scheduled job is an outage that happens to return a security status code. Same HTTP number, opposite meaning.
{: .prompt-warning }

---

## Why the dashboard stays green

This survives because most monitoring checks the wrong layer. A typical health check pings an endpoint and asserts a `200`:

```
GET /healthz → 200 OK   ✓  "service is up"
```

That probe is unauthenticated, or uses a long-lived monitoring token that *wasn't* rotated. So it stays green while the actual scheduled job — using a *different* credential that *was* rotated — fails every single run. The dashboard measures "is the service reachable," and the thing that broke is "can my job authenticate to it." Those are different questions, and only one of them was being asked.

This is the same family of mistake as a green CI run that proves nothing — see [Footguns That Pass CI]({% post_url 2026-06-13-footguns-that-pass-ci %}). The check is passing; it's just not checking the thing that matters.

---

## Monitor the outcome, not the liveness

The fix is to assert on what the automation is *supposed to accomplish*, not on whether its dependencies are pingable:

- **Check the job's own result.** Did the scheduled run complete and produce its output (a posted plan, a written row, a sent report)? An endpoint being up is necessary, not sufficient.
- **Alert on a run-failure streak.** One 403 might be a blip; N consecutive failures is a dead integration. Hundreds of silent errors should never accumulate unnoticed — the *streak* is the signal.
- **Probe with the job's real credential**, or with one rotated on the same schedule. A health check using a credential that never expires can't detect a credential-expiry failure.
- **When a secret is rotated, enumerate every consumer.** This whole incident is a rotation that updated the secret in one place and not in the scheduled job. Rotation is a fan-out operation; treat the consumer list as part of the rotation.

---

## The portable lesson

A status code describes *one* request's fate, not the system's health. Before you let a 403 (or any error) close an investigation, ask the two questions that the code alone can't answer: **whose** credential failed, and **what** did the failure stop from happening. "Auth rejected something, therefore auth is fine" skips both — and skipping both is how a job dies in plain sight behind a green dashboard.

---

## Related Posts

- [Footguns That Pass CI]({% post_url 2026-06-13-footguns-that-pass-ci %}) — a passing check that doesn't check the right thing
- [It Said "Connection Refused"]({% post_url 2026-06-13-it-said-connection-refused %}) — don't read an error as a fact about the data
- [The Empty Sync That Wiped the Dashboard]({% post_url 2026-06-29-empty-sync-wipes-data %}) — another job that "succeeded" into a silent failure
