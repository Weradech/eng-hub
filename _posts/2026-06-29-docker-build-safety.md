---
title: "Docker Build Safety: Commit Before You Rebuild"
date: 2026-06-29 09:00:00 +0700
categories: [Infrastructure, DevOps]
tags: [docker, deployment, compose, safety, devops]
---

> **TL;DR** — `docker compose build` overwrites the running image in place. If you have uncommitted changes in the container, they are gone the moment the build finishes. The safe order is: commit → build → verify → `up --no-build`.

---

![docker compose build overwrites the running image from disk; commit then build, verify, then up --no-build to deploy exactly what you verified]({{ "/assets/img/2026-06-29/docker-build-safety.svg" | relative_url }})
_Code that isn&#8217;t committed doesn&#8217;t exist: commit first, then build, verify the image, and `up --no-build`._

## The Trap

You make a hotfix directly in the running container (editing a file, pip-installing a package). The fix works. You then run `docker compose build` to "lock it in". You've just destroyed the fix.

`docker compose build` does not know about your running container. It builds a fresh image from the Dockerfile and the context on disk. If the fix never made it into the context, it doesn't make it into the image.

```
Running container          Build context (on disk)
─────────────────          ────────────────────────
app.py (patched) ──✗──X   app.py (original)
                                  │
                               docker build
                                  │
                           Image: app.py (original) ← overwrites tag
```

After `up --force-recreate`, the patch is gone.

---

## The Safe Order

```bash
# 1. Commit whatever is on disk (the source of truth)
git add -p && git commit -m "fix: ..."

# 2. Build with --no-cache so stale layers don't hide the change
docker compose build --no-cache <service>

# 3. Verify the image contains the fix before touching the running container
docker run --rm <image>:<tag> grep "fix_marker" /app/app.py

# 4. Only now swap the running container
docker compose up -d --no-build <service>
```

`--no-build` on `up` is the safety flag: if the image doesn't exist yet, the command fails rather than building silently. Use it every time after a manual `build`.

---

## The `$$` Dollar-Escape Gotcha (Compose ≥ 2.37)

Docker Compose 2.37 changed how `env_file` values are processed. A single `$` in an env file is now treated as the start of a variable substitution. If the value contains a literal dollar sign (passwords, connection strings), it silently becomes empty.

```yaml
# docker-compose.yml
env_file:
  - .env
```

```bash
# .env — works in compose <2.37, breaks in ≥2.37
DB_PASS=abc$123

# Fix: escape with $$
DB_PASS=abc$$123
```

Check your compose version after any Docker Desktop update:

```bash
docker compose version
```

If you see `2.37+` and your services start misbehaving with auth errors after an upgrade, this is the first thing to audit.

---

## Checklist Before Any Rebuild

- [ ] `git status` — no uncommitted changes in the service directory
- [ ] Build uses `--no-cache` if you changed a dependency or base image
- [ ] Verify the new image with a one-shot `docker run --rm` before recreating
- [ ] Use `up -d --no-build` after a manual build to prevent accidental double-build
- [ ] Check `.env` for unescaped `$` if upgrading Compose

---

## Why This Matters at Scale

In a high-mix low-volume manufacturing environment, deployments often happen under pressure — a jig is down, a production line is waiting. The temptation is to patch-in-place and "fix git later." The "later" that never comes is exactly when the next rebuild wipes the fix and the line goes down again.

The rule is: **code that isn't committed doesn't exist**. Build discipline is how you avoid fixing the same problem twice.
