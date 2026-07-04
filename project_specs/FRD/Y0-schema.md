---

## Database Schema

**Owned by:** F1 (Database Schema & Connectivity)  
**Used by:** F2 (Notes REST API)  
**Database:** PostgreSQL  

---

### §Notes — `notes` Table

This is the sole table in QuickNotes v1. All note data is stored here.

#### DDL

```sql
-- Enable pgcrypto for UUID generation (if not already enabled)
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Notes table — idempotent: safe to run on every startup
CREATE TABLE IF NOT EXISTS notes (
  id          UUID          NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  title       TEXT,                          -- nullable; NULL when not provided
  body        TEXT          NOT NULL,        -- required; never null or empty
  created_at  TIMESTAMPTZ   NOT NULL DEFAULT now(),
  updated_at  TIMESTAMPTZ   NOT NULL DEFAULT now()
);
```

#### Column Specifications

| Column | Type | Nullable | Default | Notes |
|--------|------|----------|---------|-------|
| `id` | `UUID` | NO | `gen_random_uuid()` | Primary key; auto-generated on insert |
| `title` | `TEXT` | YES | `NULL` | Optional; stored as NULL if not provided or empty |
| `body` | `TEXT` | NO | — | Required; validated non-empty at API layer before insert |
| `created_at` | `TIMESTAMPTZ` | NO | `now()` | Set on insert; never updated |
| `updated_at` | `TIMESTAMPTZ` | NO | `now()` | Set on insert; bumped to `now()` on every `UPDATE` |

#### Indexes

```sql
-- Default B-tree index on primary key (implicit via PRIMARY KEY constraint)
-- Ordering index for the list endpoint (GET /api/notes → ORDER BY created_at DESC)
CREATE INDEX IF NOT EXISTS idx_notes_created_at ON notes (created_at DESC);
```

#### Update Trigger (optional but recommended)

If using raw SQL without an ORM, either set `updated_at = now()` explicitly in every `UPDATE` statement (preferred for clarity in Next.js Route Handlers) or use a trigger:

```sql
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER notes_updated_at
  BEFORE UPDATE ON notes
  FOR EACH ROW
  EXECUTE FUNCTION set_updated_at();
```

**Recommendation for v1:** Set `updated_at = now()` explicitly in the `UPDATE` SQL in the Route Handler — simpler, no trigger dependency.

#### Constraints

- `id` is always generated server-side; clients never provide it.
- `body` is enforced NOT NULL at the database level AND validated non-empty at the application level (before the SQL statement executes).
- `title` `NULL` and `title = ''` (empty string) are semantically equivalent in the UI (both render as "Untitled") but the database stores `NULL` — empty strings are normalised to `NULL` by the API layer.

#### Migration Strategy

- Use `CREATE TABLE IF NOT EXISTS` — idempotent, safe to run on every app startup.
- Use `CREATE EXTENSION IF NOT EXISTS pgcrypto` — idempotent.
- Use `CREATE INDEX IF NOT EXISTS` — idempotent.
- If using Prisma: run `prisma migrate deploy` or `prisma db push` at startup; both are idempotent.
- Never run migrations via Docker Compose — always natively via `DATABASE_URL`.

---

### §Environment

No additional tables required in v1. `DATABASE_URL` is an environment variable, not a stored config.

---
