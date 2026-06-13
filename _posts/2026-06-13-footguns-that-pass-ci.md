---
title: "Four Footguns That Pass CI and Fail in Production"
date: 2026-06-13 18:00:00 +0700
categories: [System, Engineering]
tags: [engineering, gotchas, python, sqlalchemy, web-platform, qa]
---

> **TL;DR** — Four bugs a green build waves straight through, because the compiler, the type-checker, and the test runner are all structurally unable to see them. Each one shipped to production; each one has a one-line defense.

---

## 1. The `::` that ate the bind parameter

Writing a raw query with a PostgreSQL cast looks harmless and *compiles* fine:

```python
text("SELECT * FROM t WHERE qty > :threshold::numeric")
```

But SQLAlchemy's `text()` parser reads `:threshold` **and then** `:numeric` as two named bind parameters — `::` is just two colons to it. At runtime you get a missing-parameter error, or a silently wrong bind.

**Defense:** use the SQL-standard cast and avoid `::` entirely:

```python
text("SELECT * FROM t WHERE qty > CAST(:threshold AS numeric)")
```

`CAST(... AS ...)` also lets the driver type a bare `NULL` bind. Catch this class with a smoke test that **executes** the real SQL — not one that only *builds* the statement.

---

## 2. The bare `logger` that hid the real error

A module imports its logger as `_logger`. Someone writes an exception handler that reaches for `logger`:

```python
try:
    do_work()
except SomeError:
    logger.warning("falling back")   # NameError: 'logger' is not defined
    return fallback()
```

There is no `logger` in scope. The handler raises `NameError`, which **masks the original exception** — and a request that should have returned a clean 4xx returns a 500 with a misleading traceback. The bug isn't the fallback; it's that the safety net had a hole exactly where you needed it.

**Defense:** this is precisely what `ruff`'s `F821` (undefined name) catches. Wire it into the pre-push gate. A static check is free; a masked production error costs you an evening of chasing the wrong stack trace.

> Code inside an `except` block is the least-tested code you own — it only runs when something already went wrong. Lint it harder, not softer.
{: .prompt-warning }

---

## 3. LAN over plain HTTP is not a "secure context"

`crypto.randomUUID()`, the async Clipboard API, service workers and friends are only defined in a **secure context** — HTTPS or `localhost`. Open the same app over a LAN IP on plain HTTP:

```
http://10.0.0.5/
```

…and `crypto.randomUUID` is `undefined`. The page throws on first use and the screen goes blank. It worked on your laptop because `localhost` *is* a secure context — so the gap never shows up until a colleague opens it from another machine.

**Defense:** feature-detect every secure-context-only API and provide a fallback:

```js
const id = crypto?.randomUUID?.() ?? fallbackUuid();
```

Don't assume a modern browser hands you the modern API — the *context*, not the browser version, decides.

---

## 4. "The build passed" is not "it renders"

`tsc` clean and tests green tell you the code *type-checks* and the *logic* holds. They say nothing about whether a chart actually drew, or whether a flex child collapsed to zero width. A passing build with a blank panel is the most common "but it works for me" gap there is.

**Defense:** render-verify. Drive the real page in a headless browser, screenshot it, and diff against the intent — *before* you say "done."

A real example: a chart silently rendered empty until its grid track was given an explicit non-zero size and its animation was disabled under a virtual clock. Nothing about that surfaces in a unit test — only a pixel does.

> Build-green is a claim about the *source*. Render-verify is evidence about the *product*. Ship on evidence.
{: .prompt-tip }

---

## The common thread

None of these are exotic. They share one property: **the tool you trusted to catch bugs was structurally unable to see this class of bug.** The type-checker can't see a runtime SQL parser. The test runner can't see a blank pixel. The compiler can't see a missing browser API.

The fix is never "be more careful." It's "add the check that *can* see it" — execute the real SQL, lint the names, feature-detect the API, screenshot the page.

---

## Related Posts

- [RCA for Software — Gather Symptoms Before You Touch the Keyboard]({% post_url 2026-05-27-rca-for-software-symptoms-before-hypothesis %}) — what to do when one of these slips through anyway
