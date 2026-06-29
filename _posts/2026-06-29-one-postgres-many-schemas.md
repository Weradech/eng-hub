---
title: "One Postgres, Many Schemas: Why We Didn't Do Database-Per-Service"
date: 2026-06-29 17:00:00 +0700
categories: [System, Architecture]
tags: [postgres, architecture, schemas, microservices, database, devops]
---

> **TL;DR** — The microservices playbook says one database per service. For a small team running a handful of internal systems on one server, that advice imports operational overhead you don't have the staff to pay for. A single Postgres instance with one schema per domain gives you most of the isolation, far less ops burden, and easy cross-domain reads — as long as you respect the schema boundaries you drew.

---

![Database-per-service multiplies ops overhead; one Postgres instance with a schema per domain keeps it to one]({{ "/assets/img/2026-06-29/one-postgres-many-schemas.svg" | relative_url }})
_Database-per-service multiplies the ops you maintain; one instance with a schema per domain keeps it to one._

## The instinct, and why it doesn't fit a small team

"Database per service" exists to let independent teams deploy, scale, and back up independently without coordinating. That's the right call when you have many teams and real scale.

A small in-house platform usually has the opposite situation: a handful of related systems — warehouse, materials planning, BOM, manufacturing execution, RFQ — all run by **the same one or two people**, on **one server**, mostly reading **each other's data**. There, database-per-service multiplies the parts you maintain without buying the independence those parts are for:

- N separate connection pools to size and monitor
- N backup/restore jobs to schedule and *test*
- N sets of credentials to rotate
- cross-system reporting that now needs an ETL or an API hop, because you can't `JOIN` across databases

You pay the full operational price of microservices to get a benefit (independent teams scaling independently) you don't currently need.

---

## The middle ground: one instance, one schema per domain

A single Postgres instance can hold many [schemas](https://www.postgresql.org/docs/current/ddl-schemas.html) — independent namespaces inside one database. One schema per domain:

```
productiondb (one Postgres instance)
├── public      → WMS tables + the Odoo mirror + supplier (SRM) tables
├── mrp         → materials planning
├── bom         → bill-of-materials
├── mes_core    → manufacturing execution
└── rfq         → quoting
```

(A system with very different needs can still sit outside — a lightweight CRM on its own SQLite file, for instance. The point isn't "everything in one place," it's "don't split what you can't afford to operate split.")

What this buys you:

| Concern | Database-per-service | One instance, many schemas |
|---------|----------------------|----------------------------|
| Cross-domain read | ETL / API hop | plain `JOIN` across schemas |
| Backup / restore | N jobs, N restore drills | 1 job, 1 restore drill |
| Connection pooling | N pools to tune | 1 pool to tune |
| Credential rotation | N rotations | 1 (but see the caveat below) |
| Logical isolation | strong (separate DBs) | good (separate schemas + grants) |
| Independent scaling | yes | no — shared instance resources |

For an internal platform, the right-hand column is almost always the better trade until one domain genuinely outgrows the shared instance.

---

## The boundaries you must still respect

Schemas give you *namespaces*, not *walls*. The discipline that keeps this from rotting into a single tangled database:

**Don't reach across schemas to write.** A service owns its schema and reads others through well-defined views or query paths — it does not `INSERT` into another domain's tables. This is [co-master ownership]({% post_url 2026-06-29-wms-odoo-comaster %}) applied to schemas: one writer per table, everyone else reads.

**Set `search_path` explicitly.** Relying on the default `search_path` is how a query meant for `mrp.orders` silently hits `public.orders`. Qualify table names, or set the path per connection — don't let resolution be ambient.

**Watch the shared-role footgun.** The flip side of "one instance, one credential to rotate" is that one leaked credential can touch *every* schema. If multiple containers share a single database role, rotating it is an all-or-nothing event — see [Rotating a Shared Database Password]({% post_url 2026-06-29-rotate-shared-db-password %}). Per-schema roles with scoped grants are the cleaner long-term shape; the convenience of one shared role has a real blast radius.

> "One database, many schemas" is not "one database, no boundaries." The instance is shared; the *ownership* is not. The moment two services both write the same table, you've lost the isolation that made the schema split worth drawing.
{: .prompt-warning }

---

## When you *should* split out a real database

The schema approach has a ceiling. Move a domain to its own instance when:

- **One domain's load threatens the others.** A reporting-heavy domain running giant scans can starve the connection pool everyone shares. Noisy-neighbour problems are the classic signal to physically separate.
- **A domain needs a different durability or version profile** — different backup cadence, different Postgres version, a different region.
- **Ownership genuinely diverges** — a separate team takes over a domain and needs to deploy and migrate on its own cadence.

Until one of those is true, splitting is premature. You'd be taking on N-way backup drills and credential rotation to solve a scaling problem you don't have yet.

---

## The reframing

Database-per-service answers "how do many teams stay independent at scale?" A small internal platform is asking a different question: "how do one or two people run several related systems reliably without drowning in ops?" The schema-per-domain shape answers *that* question, and you can always promote a hot domain to its own instance the day it earns it.

Start with the boundaries you can afford to operate. Draw them as schemas with clear ownership, and split to separate databases only when a concrete pressure — load, durability, or org structure — makes the shared instance the bottleneck.

---

## Related Posts

- [Co-Master Architecture]({% post_url 2026-06-29-wms-odoo-comaster %}) — single-owner discipline, the rule that keeps shared schemas honest
- [Rotating a Shared Database Password]({% post_url 2026-06-29-rotate-shared-db-password %}) — the blast radius of the one shared role this design implies
- [Alembic: Treat Revisions as a Contract]({% post_url 2026-06-29-alembic-contract %}) — migrating schemas that live in one instance
