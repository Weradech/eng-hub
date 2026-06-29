---
title: "Rotating a Shared Database Password With Zero Downtime"
date: 2026-06-29 16:30:00 +0700
categories: [Infrastructure, Security]
tags: [security, postgres, secrets, rotation, docker, devops]
---

> **TL;DR** — When several containers share one database role, the password is a single point of both failure and exposure. Change it naively and every service drops at once. This is the playbook for rotating a leaked shared credential without an outage — plus the `docker compose` escaping gotcha that bites you mid-rotation.

---

## The setup that makes rotation scary

A common small-platform shape: one Postgres role (say `app_user`), and half a dozen containers — API, worker, scheduler, a couple of sibling services — all connecting with that one credential. It's convenient. It also means:

- **A leak is total.** If that password lands in a log, a committed `.env`, or a screenshot, it's not one service exposed — it's everything that role can touch. (And with [one instance, many schemas]({% post_url 2026-06-29-one-postgres-many-schemas %}), that's potentially every domain.)
- **A naive rotation is an outage.** `ALTER ROLE app_user PASSWORD '...'` takes effect immediately. Every container still holding the old password fails its next connection. Change it in the database before the containers know, and you've taken the whole platform down.

So the password gets left alone "because rotating it is risky" — which is exactly how a leaked credential stays live for months.

---

## When it leaks: rotate first, investigate second

If the password has actually leaked — in a persistent log, in git history, anywhere durable — the priority order is:

1. **Rotate now.** A leaked credential is live until changed. Don't wait for the post-mortem.
2. **Purge it from where it leaked** (log, history) so it doesn't re-leak.
3. **Then** figure out how it got there and stop the source.

The rotation has to be doable *quickly and safely*, or step 1 keeps getting deferred. That's what the rest of this post is for.

> Never write a live secret into anything durable — a committed file, a persistent log, a chat transcript. Once it's in git history it's effectively public even after you delete the file. If it lands somewhere durable, treat it as compromised and rotate; don't rationalise that "the repo is private."
{: .prompt-warning }

---

## The zero-downtime rotation

The clean way to rotate without dropping connections is to **never have a moment where the database and the running containers disagree about the password.** Two approaches:

### Option A — coordinated rolling restart (single shared role)

If you're keeping the single shared role, the safe sequence is:

```
1. Update the secret in the source of truth (env_file / secrets manager)
   — but DON'T change it in Postgres yet.
2. ALTER ROLE app_user PASSWORD 'new'   — now the DB expects 'new'.
3. Immediately roll the containers one by one:
     recreate container → it reads 'new' from env_file → reconnects.
   Existing open connections on 'old' keep working until recycled,
   so there's no hard cutover for in-flight queries.
4. Verify each container reconnected before moving to the next.
```

The window where a *new* connection could fail is the few seconds between step 2 and each container's recreate — small, and bounded per service rather than global. Roll them sequentially and watch each come back healthy.

### Option B — dual-credential cutover (the cleaner shape)

If you can, split the single role into per-service roles, or use Postgres's ability to grant the same access to a second role:

```
1. CREATE ROLE app_user_v2 with the same grants.
2. Point services at app_user_v2 one at a time, recreate, verify.
3. When nothing uses app_user, REVOKE / DROP it.
```

Now there's *no* window where anything is broken — the old credential stays valid until the last service has moved off it. This is the same idea as a blue-green deploy, applied to a credential. It's more setup, but it turns rotation from a held-breath event into a routine one, and it shrinks the blast radius for next time.

---

## The gotcha that derails the rotation: `$` in `env_file`

Here's the one that wastes an afternoon. A strong generated password often contains `$`. Docker Compose performs variable interpolation on `env_file` values, and **a single `$` gets eaten as the start of a variable reference.**

```bash
# .env  — you set this:
DB_PASSWORD=ab$cd1234

# what the container actually receives:
DB_PASSWORD=abcd1234        # "$cd1234"... "$cd" expanded to empty!
```

The database has `ab$cd1234`; the container connects with `abcd1234`; authentication fails — and it looks like you typed the password wrong, not like an escaping bug. The fix (Compose ≥ 2.x) is to **double the dollar sign** in the file:

```bash
# .env
DB_PASSWORD=ab$$cd1234       # $$ → literal $ → container gets ab$cd1234
```

> A leading-`$` password that "works in `psql` but fails in the container" is almost always `env_file` interpolation eating the `$`. Escape it as `$$`, or generate passwords from an alphabet that excludes `$` so the problem can't occur.
{: .prompt-tip }

---

## The portable checklist

- **A shared role means shared blast radius.** Prefer per-service roles with scoped grants; if you must share, document who connects with it.
- **Rotate the instant a secret hits anything durable.** Leaked = live until changed.
- **Update the source of truth before the database, then roll containers** — never flip the DB and leave containers on the old secret.
- **Prefer a dual-credential cutover** when you can; it removes the failure window entirely.
- **Escape `$` as `$$` in `env_file`**, or avoid `$` in generated passwords altogether.
- **Verify each service reconnects** before moving to the next — a half-rotated platform is worse than an un-rotated one.

---

## Related Posts

- [One Postgres, Many Schemas]({% post_url 2026-06-29-one-postgres-many-schemas %}) — why the shared role exists, and its blast radius
- [Docker Build Safety]({% post_url 2026-06-29-docker-build-safety %}) — recreating containers without losing uncommitted state
- [PM2 Gotchas]({% post_url 2026-06-29-pm2-gotchas %}) — `restart` caches stale env; `delete + start` is the reliable pattern
