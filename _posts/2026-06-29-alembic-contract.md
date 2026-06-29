---
title: "Alembic: Treat Revisions as a Contract, Not a Convenience"
date: 2026-06-29 09:40:00 +0700
categories: [Infrastructure, Database]
tags: [alembic, sqlalchemy, python, database, migrations, reliability]
---

> **TL;DR** — Alembic migrations are a contract between your code and the database schema. Two failure modes break this contract silently: a model that isn't imported is never migrated (`autogenerate` just doesn't see it), and a `text()` query with PostgreSQL-style `::` casts crashes at runtime even though it works fine in psql. Both are easy to introduce and hard to notice.

---

## Failure Mode 1: The Model That Doesn't Exist

Alembic's `autogenerate` works by importing your SQLAlchemy models and comparing them against the live database. If a model isn't imported, `autogenerate` treats it as if it doesn't exist — and generates no migration for it.

```python
# models/inventory.py
class StockLedger(Base):
    __tablename__ = "stock_ledger"
    id: Mapped[int] = mapped_column(primary_key=True)
    # ...
```

```python
# models/__init__.py  ← the culprit
from .product import Product
from .order import Order
# StockLedger is NOT imported here
```

```bash
alembic revision --autogenerate -m "add stock_ledger"
# Generates an empty migration: "No changes detected"
```

The table is never created. The first time the app tries to write to `stock_ledger`, it fails with `UndefinedTable`. If the app is new and the table is small, you might not notice until you're in production.

**The fix:** every model must be imported in the package `__init__.py` before `env.py` calls `target_metadata`.

```python
# models/__init__.py
from .product import Product
from .order import Order
from .inventory import StockLedger   # ← add this
```

```python
# alembic/env.py
from myapp import models  # this import must trigger __init__.py
target_metadata = models.Base.metadata
```

**Verification before any `autogenerate`:**

```bash
python -c "from myapp.models import Base; print(Base.metadata.tables.keys())"
# stock_ledger must appear in the output
```

---

## Failure Mode 2: PostgreSQL `::` Cast in `text()`

PostgreSQL supports the `::` syntax for type casting:

```sql
SELECT id::text FROM orders WHERE created_at > '2026-01-01'::timestamp
```

This works perfectly in `psql`. It also works in raw `psycopg2`. But SQLAlchemy's `text()` function passes the string through its own parameter parser, which interprets `::` as a special token in some contexts.

```python
# Breaks at runtime with SQLAlchemy text()
from sqlalchemy import text

result = db.execute(text("""
    SELECT amount::numeric FROM ledger WHERE ts > :cutoff::timestamp
"""), {"cutoff": "2026-01-01"})
```

The error is not a syntax error in your Python — it's a runtime exception that only surfaces when the query is executed. In testing environments that skip the database (`@pytest.mark.skip("no pg")`), this bug never fires.

**The fix:** use `CAST()` instead of `::`.

```python
# Works reliably with SQLAlchemy text()
result = db.execute(text("""
    SELECT CAST(amount AS numeric) FROM ledger
    WHERE ts > CAST(:cutoff AS timestamp)
"""), {"cutoff": "2026-01-01"})
```

`CAST()` is standard SQL and SQLAlchemy's parser handles it without ambiguity.

---

## Revisions Are a Contract

Beyond these two failure modes, the broader principle is: **an Alembic revision is a contract between the codebase and the database**. Treating it otherwise creates drift.

### What breaks the contract

| Action | Why it breaks |
|--------|--------------|
| Editing a migration after it's been applied | DB state diverges from migration history |
| Applying migrations out of order | Dependencies between revisions are violated |
| Deleting a migration without a compensating one | `alembic heads` disagrees with DB `alembic_version` table |
| Adding columns directly in psql | Schema has columns Alembic doesn't know about |
| Running `autogenerate` without all models imported | Missing tables/columns are never migrated |

### Pre-deploy verification

Before every deployment that includes migrations:

```bash
# 1. Verify alembic heads matches what's in the DB
alembic heads          # what the code expects
alembic current        # what the DB currently has

# 2. Preview the migration SQL before applying
alembic upgrade head --sql   # prints SQL, does not execute

# 3. Apply
alembic upgrade head

# 4. Confirm
alembic current        # should match alembic heads
```

### Schema drift detection

Add this to your CI or startup check:

```python
from alembic.runtime.migration import MigrationContext
from alembic.script import ScriptDirectory
from sqlalchemy import create_engine

def check_migrations_applied(db_url: str, alembic_ini: str) -> bool:
    engine = create_engine(db_url)
    config = Config(alembic_ini)
    script = ScriptDirectory.from_config(config)
    
    with engine.connect() as conn:
        context = MigrationContext.configure(conn)
        current = set(context.get_current_heads())
        expected = set(script.get_heads())
        
    return current == expected
```

If this returns `False` at startup, abort — the service is running against the wrong schema.

---

## Quick Reference

```bash
# Check what models alembic can see
python -c "from myapp.models import Base; print(sorted(Base.metadata.tables))"

# Generate migration (only after confirming models are imported)
alembic revision --autogenerate -m "describe the change"

# Preview SQL without running
alembic upgrade head --sql

# Apply
alembic upgrade head

# Rollback one step
alembic downgrade -1

# Check current DB state
alembic current

# Show migration history
alembic history --verbose
```
