---

## F01: Database Schema & Connectivity

**Description:** This feature establishes the PostgreSQL connection and ensures the `notes` table exists with the correct schema before any API or UI request is served. The app reads `DATABASE_URL` from the environment, fails fast with a descriptive error if it is absent, and runs idempotent migrations at startup — without Docker or Compose.

**Priority:** P0 — Critical. All data features (F2–F5) depend on this.

**Depends On:** F0 (scaffold must exist for startup lifecycle hooks).

---

### Terminology

- **DATABASE_URL:** PostgreSQL connection string in the format `postgresql://user:password@host:port/dbname`. Injected by the runtime environment.
- **PIVOTA_DB_MODE:** Optional environment variable. When set to `sidecar`, the app connects directly to the provisioned database and runs migrations natively (no Compose).
- **Idempotent migration:** A DDL statement safe to re-execute on every startup (e.g. `CREATE TABLE IF NOT EXISTS`, `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`).
- **Fail fast:** The app exits immediately (non-zero exit code) with a human-readable error message if a required environment variable is missing or the database is unreachable at startup.
- **pg client:** The `pg` (node-postgres) npm package used to execute raw SQL queries against PostgreSQL.
- **Prisma:** An alternative ORM/query-builder; if used, migrations run via `prisma migrate deploy` or `prisma db push` at startup against `DATABASE_URL`.

---

### Sub-features

- Environment variable validation at startup (`DATABASE_URL` required)
- PostgreSQL connection establishment using `pg` or Prisma
- Idempotent `notes` table creation/migration at startup
- Sidecar mode support via `PIVOTA_DB_MODE`
- Connection pool configuration (reuse across requests)

---

### Process

1. Application starts (`npm run dev` / `npm start`).
2. Startup code reads `process.env.DATABASE_URL`.
3. If `DATABASE_URL` is `undefined` or empty string, the process logs `"FATAL: DATABASE_URL environment variable is not set."` and exits with code 1.
4. If `PIVOTA_DB_MODE` is set to `sidecar`, the app proceeds in direct-connect mode (no behavioural difference in v1 — same connection flow).
5. A PostgreSQL client/pool is initialised with `DATABASE_URL`.
6. The startup migration runs the DDL in `Y0-schema.md` §Notes (idempotent `CREATE TABLE IF NOT EXISTS notes ...`).
7. If the migration fails (e.g. permission denied, connection refused), the process logs the error with context and exits with code 1.
8. On successful migration, the connection pool is made available to all Route Handlers (singleton pattern).
9. The dev server continues and begins accepting HTTP requests.

---

### Inputs

- `DATABASE_URL` (environment variable, string, required): PostgreSQL connection string.
- `PIVOTA_DB_MODE` (environment variable, string, optional): When set, confirms sidecar/native connection mode. No change to connection logic in v1.

---

### Outputs

- **Success:** PostgreSQL connection pool available application-wide; `notes` table exists and has the correct schema.
- **Failure (missing env):** Process exits with code 1; log message: `"FATAL: DATABASE_URL environment variable is not set."`
- **Failure (connection error):** Process exits with code 1; log message includes the PostgreSQL error string and `DATABASE_URL` host (not the full URL with credentials — strip password for safety).

---

### Validation

- `DATABASE_URL` must be present (non-null, non-empty). Check at module initialisation time, not lazily on first request.
- The migration DDL must be idempotent: safe to run on every startup, including on a database that already has the `notes` table.
- The connection pool must be a singleton — not a new connection per request.
- All columns must match the schema in `Y0-schema.md` exactly:
  - `id`: UUID primary key (or serial — see schema notes)
  - `title`: text, nullable
  - `body`: text, not null
  - `created_at`: timestamptz, default `now()`
  - `updated_at`: timestamptz, default `now()`, updated on every `PUT`

---

### Error States

| Scenario | Behaviour | Log Message |
|----------|-----------|-------------|
| `DATABASE_URL` absent | Exit code 1 immediately | `"FATAL: DATABASE_URL environment variable is not set."` |
| Database unreachable at startup | Exit code 1 after connection timeout | `"FATAL: Cannot connect to database: <pg error>"` |
| Migration fails (permissions) | Exit code 1 | `"FATAL: Migration failed: <pg error>"` |
| `notes` table already exists (normal restart) | No error — `IF NOT EXISTS` is a no-op | None |
| `DATABASE_URL` set but malformed URL | Exit code 1 with pg parse error | `"FATAL: Cannot connect to database: <pg error>"` |

---

### API Surface (this feature)

This feature has no direct API endpoints. It underpins all data-returning endpoints in F2 (`/api/notes`).

---

### Schema Surface (this feature)

Defines and owns the `notes` table. Full DDL: see `Y0-schema.md` §Notes.

**Column summary:**
- `id` — `UUID` primary key, default `gen_random_uuid()`
- `title` — `TEXT`, nullable
- `body` — `TEXT`, not null
- `created_at` — `TIMESTAMPTZ`, default `now()`
- `updated_at` — `TIMESTAMPTZ`, default `now()`

---
