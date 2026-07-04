# Technical Architecture Document
# QuickNotes

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Version:** 1.0  
**Date:** 2026-07-04  
**Status:** Active  
**Based On:** PRD-quicknotes2.md v1.0, FRD-quicknotes2.md v1.0  

---

## Table of Contents

1. [Architectural Overview](#1-architectural-overview)
2. [Component Architecture](#2-component-architecture)
3. [Data Model](#3-data-model)
4. [API Design](#4-api-design)
5. [Security Architecture](#5-security-architecture)
6. [Technology Stack](#6-technology-stack)
7. [Integration Points](#7-integration-points)

---

## 1. Architectural Overview

### 1.1 Architecture Pattern

QuickNotes uses a **Monolithic Single-Process** architecture: one Next.js process on port 3000 serves both the React UI (via SSR/RSC) and the REST API (via Route Handlers). There is no separate API server, no microservices boundary, and no message queue. This is the correct pattern for a single-user MVP with a well-scoped feature set.

**Key architectural decisions:**

| Decision | Rationale |
|----------|-----------|
| Monolithic Next.js process | Simplest deployment unit; UI and API share one port, one build, one process |
| App Router (not Pages Router) | Modern Next.js pattern required by spec; collocates route handlers close to their data |
| Direct PostgreSQL via `pg` or Prisma | Avoids heavy ORMs; thin layer gives full SQL control and predictable query behaviour |
| Idempotent startup migrations | Safe to restart at any time; no separate migration runner process or Docker Compose |
| No authentication layer | Single implicit user; eliminates session/token complexity from the v1 surface |

### 1.2 Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                    Browser (Evergreen)                           │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  React SPA (hydrated from SSR)                          │    │
│  │                                                         │    │
│  │  ┌──────────────┐   ┌──────────────────────────────┐   │    │
│  │  │  ComposeBox  │   │  NoteList                    │   │    │
│  │  │  (F3)        │   │  ┌────────────────────────┐  │   │    │
│  │  │              │   │  │  NoteCard (F4/F5)       │  │   │    │
│  │  │  title input │   │  │  read / edit / delete   │  │   │    │
│  │  │  body area   │   │  └────────────────────────┘  │   │    │
│  │  │  Add Note btn│   │  empty state                  │   │    │
│  │  └──────┬───────┘   └──────────────┬───────────────┘   │    │
│  └─────────┼──────────────────────────┼───────────────────┘    │
│            │  fetch()                 │  fetch()                │
└────────────┼──────────────────────────┼────────────────────────┘
             │  HTTP/JSON               │  HTTP/JSON
             ▼                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                Next.js Process  (0.0.0.0:3000)                  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Route Handlers  (app/api/)                              │   │
│  │                                                          │   │
│  │   GET  /health          →  HealthHandler (no DB)         │   │
│  │   GET  /api/notes       →  NotesCollectionHandler        │   │
│  │   POST /api/notes       →  NotesCollectionHandler        │   │
│  │   GET  /api/notes/[id]  →  NoteResourceHandler           │   │
│  │   PUT  /api/notes/[id]  →  NoteResourceHandler           │   │
│  │   DELETE /api/notes/[id]→  NoteResourceHandler           │   │
│  └────────────────────────────┬─────────────────────────────┘   │
│                               │                                  │
│  ┌────────────────────────────▼─────────────────────────────┐   │
│  │  DB Layer  (lib/db.ts)                                   │   │
│  │                                                          │   │
│  │   Singleton Pool  ──►  Startup Migration                 │   │
│  │   (pg.Pool / Prisma)    (CREATE TABLE IF NOT EXISTS)     │   │
│  └────────────────────────────┬─────────────────────────────┘   │
└───────────────────────────────┼──────────────────────────────────┘
                                │  PostgreSQL wire protocol
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                PostgreSQL  (DATABASE_URL)                        │
│                                                                  │
│   notes table                                                    │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  id · title · body · created_at · updated_at             │  │
│   └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### 1.3 Deployment Topology

```
┌─────────────────────────────────────────────────┐
│  Sandbox / Preview Environment                  │
│                                                 │
│  ┌─────────────────────┐  ┌──────────────────┐  │
│  │  Node.js Process    │  │  PostgreSQL       │  │
│  │  next dev / start   │  │  (provisioned by  │  │
│  │  0.0.0.0:3000       │  │   environment)    │  │
│  └──────────┬──────────┘  └────────┬─────────┘  │
│             │  DATABASE_URL         │             │
│             └──────────────────────┘             │
│                                                 │
│  Environment Variables:                         │
│    DATABASE_URL  (required)                     │
│    PIVOTA_DB_MODE  (optional — sidecar)         │
└─────────────────────────────────────────────────┘
```

**Runtime model:** The environment provisions the PostgreSQL database and injects its connection string as `DATABASE_URL`. The Next.js process connects directly — no Docker, no Compose, no proxy. Migrations run at process startup before the HTTP server accepts requests.

---

## 2. Component Architecture

### 2.1 Backend Components

#### 2.1.1 DB Layer — `lib/db.ts`

**Responsibility:** Owns the singleton PostgreSQL connection pool and the startup migration sequence. All Route Handlers import this module and share a single pool.

**Behaviour:**
1. Reads `process.env.DATABASE_URL` at module initialisation.
2. If absent or empty → logs `"FATAL: DATABASE_URL environment variable is not set."` and calls `process.exit(1)`.
3. Creates a `pg.Pool` (or Prisma client) using `DATABASE_URL`.
4. Runs the idempotent migration DDL (see §3).
5. If migration fails → logs `"FATAL: Migration failed: <pg error>"` and calls `process.exit(1)`.
6. Exports the pool singleton for use by all Route Handlers.

**File structure:**
```
lib/
  db.ts          ← pool singleton + startup migration
```

#### 2.1.2 Health Route Handler — `app/api/health/route.ts`

**Responsibility:** Liveness probe endpoint. Zero database dependency.

**Behaviour:** Returns `{ "status": "ok" }` with HTTP 200 immediately. Must respond in under 100 ms. Never opens a database connection.

#### 2.1.3 Notes Collection Handler — `app/api/notes/route.ts`

**Responsibility:** Handles `GET /api/notes` (list) and `POST /api/notes` (create).

| Method | SQL | Success | Error cases |
|--------|-----|---------|-------------|
| GET | `SELECT … FROM notes ORDER BY created_at DESC` | 200 + array | 500 on DB error |
| POST | Validate body → `INSERT INTO notes … RETURNING *` | 201 + note | 400 (body empty/missing), 400 (bad JSON), 500 |

#### 2.1.4 Note Resource Handler — `app/api/notes/[id]/route.ts`

**Responsibility:** Handles `GET`, `PUT`, `DELETE` for a single note by UUID.

| Method | SQL | Success | Error cases |
|--------|-----|---------|-------------|
| GET | `SELECT … FROM notes WHERE id = $1` | 200 + note | 404 (not found), 500 |
| PUT | Validate body → `UPDATE notes SET … WHERE id = $1 RETURNING *` | 200 + note | 400 (body), 404 (not found), 500 |
| DELETE | `DELETE FROM notes WHERE id = $1 RETURNING id` | 204 (no body) | 404 (not found), 500 |

### 2.2 Frontend Components

All UI components live in `app/` and are React Client Components (interactivity via React state and `fetch`). The root page is server-rendered for initial HTML.

```
app/
  page.tsx              ← Root page (SSR shell; renders ComposeBox + NoteList)
  components/
    ComposeBox.tsx       ← F3: title input, body textarea, Add Note button
    NoteList.tsx         ← F4: fetches and renders list; owns refresh state
    NoteCard.tsx         ← F4/F5: read view + inline edit/delete
```

#### 2.2.1 `ComposeBox` (F3)

- Owns local state: `title: string`, `body: string`, `isSubmitting: boolean`, `error: string | null`.
- Renders `<label>` + `<input>` for title; `<label>` + `<textarea>` for body.
- "Add Note" `<button disabled={body.trim() === '' || isSubmitting}>`.
- On submit: `POST /api/notes` → on 201: clears form, calls `onNoteCreated()` prop (triggers `NoteList` refresh).
- On error: sets `error` string; form preserved.

#### 2.2.2 `NoteList` (F4)

- Owns state: `notes: Note[]`, `isLoading: boolean`, `error: string | null`.
- `fetchNotes()` function calls `GET /api/notes` and sets state.
- Exposes `refresh()` (alias of `fetchNotes`) to parent for cross-component coordination.
- Renders: loading indicator → error message → empty state → list of `NoteCard` elements.
- Re-fetches after create (via prop callback from ComposeBox), edit, or delete signals from NoteCard.

#### 2.2.3 `NoteCard` (F4/F5)

- Props: `note: Note`, `onRefresh: () => void`.
- Owns state: `isEditing: boolean`, `editTitle: string`, `editBody: string`, `isSaving: boolean`, `error: string | null`.
- **Read view:** title (or "Untitled"), body, `<time dateTime={note.created_at}>` relative timestamp, Edit button, Delete button.
- **Edit mode:** pre-filled `<input>` (title) + `<textarea>` (body), Save button (disabled when `editBody.trim() === ''`), Cancel button.
- **Save:** `PUT /api/notes/:id` → on 200: exits edit mode, updates card from response, calls `onRefresh()`.
- **Delete:** `window.confirm(…)` → `DELETE /api/notes/:id` → on 204: calls `onRefresh()`. Optionally optimistic removal with rollback on error.

### 2.3 Shared Types — `types/index.ts`

```typescript
export interface Note {
  id: string;          // UUID
  title: string | null;
  body: string;
  created_at: string;  // ISO 8601 UTC
  updated_at: string;  // ISO 8601 UTC
}
```

---

## 3. Data Model

### 3.1 Entity Relationship Diagram

QuickNotes v1 has a single entity. No relationships.

```
┌─────────────────────────────────────────────────┐
│                    notes                        │
├──────────────┬─────────────────────────────────┤
│  id          │  UUID  PK  NOT NULL             │
│  title       │  TEXT  nullable                 │
│  body        │  TEXT  NOT NULL                 │
│  created_at  │  TIMESTAMPTZ  NOT NULL          │
│  updated_at  │  TIMESTAMPTZ  NOT NULL          │
└──────────────┴─────────────────────────────────┘
```

### 3.2 DDL — Complete Schema

```sql
-- ============================================================
-- QuickNotes v1 — PostgreSQL Schema
-- Idempotent: safe to run on every application startup
-- ============================================================

-- Enable UUID generation (pgcrypto — available on all managed PG)
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- ============================================================
-- Table: notes
-- Sole table in QuickNotes v1.
-- All note data is stored here.
-- ============================================================
CREATE TABLE IF NOT EXISTS notes (
    id          UUID          NOT NULL DEFAULT gen_random_uuid(),
    title       TEXT,                              -- optional; NULL when not provided
    body        TEXT          NOT NULL,            -- required; validated non-empty at API layer
    created_at  TIMESTAMPTZ   NOT NULL DEFAULT now(),
    updated_at  TIMESTAMPTZ   NOT NULL DEFAULT now(),

    CONSTRAINT notes_pkey PRIMARY KEY (id)
);

-- ============================================================
-- Indexes
-- ============================================================

-- Performance index for GET /api/notes (ORDER BY created_at DESC)
CREATE INDEX IF NOT EXISTS idx_notes_created_at
    ON notes (created_at DESC);

-- ============================================================
-- Notes
-- ============================================================
-- * id is generated server-side (gen_random_uuid()); clients never supply it.
-- * body is enforced NOT NULL at DB level AND validated non-empty at API level.
-- * title NULL and title '' are semantically equivalent in the UI ("Untitled");
--   the API layer normalises empty strings to NULL before INSERT/UPDATE.
-- * updated_at is set explicitly in every UPDATE statement:
--     UPDATE notes SET …, updated_at = now() WHERE id = $n RETURNING *
--   A trigger is not required in v1 since all writes go through the Route Handler.
```

### 3.3 Column Specifications

| Column | Type | Nullable | Default | Notes |
|--------|------|----------|---------|-------|
| `id` | `UUID` | NO | `gen_random_uuid()` | Primary key; auto-generated on insert; clients never provide |
| `title` | `TEXT` | YES | `NULL` | Optional label; empty string normalised to `NULL` by API layer |
| `body` | `TEXT` | NO | — | Required content; non-empty enforced at API before SQL executes |
| `created_at` | `TIMESTAMPTZ` | NO | `now()` | Set on insert; never modified |
| `updated_at` | `TIMESTAMPTZ` | NO | `now()` | Set on insert; explicitly bumped to `now()` on every `UPDATE` |

### 3.4 Migration Strategy

| Rule | Implementation |
|------|----------------|
| Idempotent DDL | `CREATE TABLE IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`, `CREATE EXTENSION IF NOT EXISTS` |
| Startup-time execution | Migration runs in `lib/db.ts` before the HTTP server serves requests |
| Prisma alternative | If using Prisma: `prisma migrate deploy` or `prisma db push` called at startup against `DATABASE_URL` |
| Never via Compose | Migrations are native Node.js invocations — no Docker dependency |
| Fail fast on error | Migration failure → `process.exit(1)` with descriptive log |

---

## 4. API Design

### 4.1 API Overview

| Property | Value |
|----------|-------|
| Base URL (dev) | `http://localhost:3000` |
| API prefix | `/api/` |
| Protocol | HTTP/1.1 + JSON |
| Content-Type | `application/json` for all requests and responses |
| Authentication | None (single-user, no auth in v1) |
| Error format | `{ "error": "<human-readable message>" }` for all 4xx/5xx |

### 4.2 TypeScript Interfaces

```typescript
// ============================================================
// Shared Types
// ============================================================

/** A note as returned by all API endpoints */
export interface Note {
  id: string;           // UUID primary key
  title: string | null; // null when not set
  body: string;         // never null or empty
  created_at: string;   // ISO 8601 UTC string
  updated_at: string;   // ISO 8601 UTC string
}

/** Request body for POST /api/notes */
export interface CreateNoteRequest {
  title?: string | null; // optional; null/empty → stored as NULL
  body: string;          // required; non-empty after trim
}

/** Request body for PUT /api/notes/:id */
export interface UpdateNoteRequest {
  title?: string | null; // optional; same normalisation as create
  body: string;          // required; non-empty after trim
}

/** Shape of all API error responses (4xx / 5xx) */
export interface ApiError {
  error: string;
}

/** GET /health response */
export interface HealthResponse {
  status: 'ok';
}
```

### 4.3 Endpoint Catalog

#### GET /health

| Property | Value |
|----------|-------|
| Method | GET |
| Path | `/health` |
| Auth | None |
| DB dependency | None |
| SLA | < 100 ms |

**Response — 200 OK:**
```json
{ "status": "ok" }
```

---

#### GET /api/notes

| Property | Value |
|----------|-------|
| Method | GET |
| Path | `/api/notes` |
| Auth | None |
| Query params | None |

**Response — 200 OK:** JSON array of `Note` objects, ordered `created_at DESC`. Returns `[]` when no notes exist.

```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "title": "My Title",
    "body": "Note body text.",
    "created_at": "2026-07-04T12:00:00.000Z",
    "updated_at": "2026-07-04T12:00:00.000Z"
  }
]
```

**SQL:**
```sql
SELECT id, title, body, created_at, updated_at
FROM notes
ORDER BY created_at DESC;
```

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 500 | `{ "error": "Internal server error" }` | Database query failure |

---

#### POST /api/notes

| Property | Value |
|----------|-------|
| Method | POST |
| Path | `/api/notes` |
| Auth | None |
| Request body | `CreateNoteRequest` |

**Request body:**
```json
{ "title": "Optional Title", "body": "Required body text." }
```

**Validation:**
- `body` must be a non-null, non-empty string after `trim()` → 400 otherwise
- `title` is optional; `null`, missing, or empty string after `trim()` → stored as `NULL`
- Request body must be valid JSON → 400 `{ "error": "Invalid JSON body" }` otherwise

**SQL:**
```sql
INSERT INTO notes (title, body)
VALUES ($1, $2)
RETURNING *;
```

**Response — 201 Created:** Full `Note` object.

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 400 | `{ "error": "body is required" }` | `body` missing, null, empty, or whitespace-only |
| 400 | `{ "error": "Invalid JSON body" }` | Request body is not valid JSON |
| 500 | `{ "error": "Internal server error" }` | Database insert failure |

---

#### GET /api/notes/:id

| Property | Value |
|----------|-------|
| Method | GET |
| Path | `/api/notes/:id` |
| Path param | `id` — UUID string |
| Auth | None |

**SQL:**
```sql
SELECT id, title, body, created_at, updated_at
FROM notes
WHERE id = $1;
```

**Response — 200 OK:** Single `Note` object.

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 404 | `{ "error": "Note not found" }` | No note with given `id` |
| 500 | `{ "error": "Internal server error" }` | Database query failure |

---

#### PUT /api/notes/:id

| Property | Value |
|----------|-------|
| Method | PUT |
| Path | `/api/notes/:id` |
| Path param | `id` — UUID string |
| Auth | None |
| Request body | `UpdateNoteRequest` |

**Validation:** Same rules as POST — `body` required and non-empty; `title` optional.

**SQL:**
```sql
UPDATE notes
SET title = $1, body = $2, updated_at = now()
WHERE id = $3
RETURNING *;
```

**Response — 200 OK:** Updated `Note` object (0 rows updated → 404).

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 400 | `{ "error": "body is required" }` | `body` missing, null, or empty |
| 400 | `{ "error": "Invalid JSON body" }` | Malformed request JSON |
| 404 | `{ "error": "Note not found" }` | No note with given `id` |
| 500 | `{ "error": "Internal server error" }` | Database update failure |

---

#### DELETE /api/notes/:id

| Property | Value |
|----------|-------|
| Method | DELETE |
| Path | `/api/notes/:id` |
| Path param | `id` — UUID string |
| Auth | None |

**SQL:**
```sql
DELETE FROM notes
WHERE id = $1
RETURNING id;
```

**Response — 204 No Content:** Empty body (0 rows deleted → 404).

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 404 | `{ "error": "Note not found" }` | No note with given `id` |
| 500 | `{ "error": "Internal server error" }` | Database delete failure |

---

### 4.4 Endpoint Summary

| Method | Path | Handler File | Success Status | Body |
|--------|------|-------------|---------------|------|
| GET | `/health` | `app/api/health/route.ts` | 200 | `{ "status": "ok" }` |
| GET | `/api/notes` | `app/api/notes/route.ts` | 200 | `Note[]` |
| POST | `/api/notes` | `app/api/notes/route.ts` | 201 | `Note` |
| GET | `/api/notes/:id` | `app/api/notes/[id]/route.ts` | 200 | `Note` |
| PUT | `/api/notes/:id` | `app/api/notes/[id]/route.ts` | 200 | `Note` |
| DELETE | `/api/notes/:id` | `app/api/notes/[id]/route.ts` | 204 | (empty) |

---

## 5. Security Architecture

### 5.1 Authentication & Authorization

QuickNotes v1 is explicitly single-user with **no authentication layer**. All API endpoints are publicly accessible on the configured port. This is an intentional scope decision; auth is listed as out-of-scope for v1 in both the PRD and PROJECT.md.

**Security posture by endpoint:**

| Endpoint | Auth | Notes |
|----------|------|-------|
| `GET /health` | None | Liveness probe only; no data exposure |
| `GET /api/notes` | None | Single-user; all notes belong to the implicit user |
| `POST /api/notes` | None | Any caller can create a note |
| `GET/PUT/DELETE /api/notes/:id` | None | Any caller with a valid UUID can read/modify/delete |

**Implication:** The app should only be exposed on a trusted network or behind a proxy with network-level access controls (e.g. sandbox firewall) in production. Multi-user or internet-facing deployments must introduce authentication before shipping.

### 5.2 Input Validation

All user-supplied data is validated at the API layer before reaching the database:

| Input | Validation Rule | Rejection |
|-------|----------------|-----------|
| `body` (POST/PUT) | Non-null string; at least one non-whitespace character after `trim()` | 400 `body is required` |
| `title` (POST/PUT) | Optional; any string or null | Normalised to `NULL`; never rejected |
| `id` (path param) | Non-empty string; existence checked via DB query | 404 if not found |
| Request body | Must be valid JSON | 400 `Invalid JSON body` |

**SQL injection prevention:** All SQL uses parameterised queries (`$1`, `$2`, etc.) via `pg` — never string interpolation. User data never touches the query string.

### 5.3 Data Protection

| Concern | Approach |
|---------|----------|
| `DATABASE_URL` credential exposure | Never logged; password stripped from any error log output (log host only) |
| Data at rest | Delegated to PostgreSQL host (environment provider's responsibility) |
| Data in transit (browser ↔ app) | HTTP in dev; TLS via reverse proxy for production deployments |
| XSS | React escapes all rendered user content by default; no `dangerouslySetInnerHTML` |
| CSRF | No auth cookies; no CSRF surface in v1 |

### 5.4 Dependency Surface

QuickNotes v1 has a minimal dependency footprint — reducing supply chain risk:

| Dependency | Role | Trust |
|------------|------|-------|
| `next` | Framework | Vercel-maintained; high trust |
| `react` / `react-dom` | UI runtime | Meta-maintained; high trust |
| `pg` (node-postgres) | DB client | Widely used; actively maintained |
| `typescript` | Type system | Microsoft-maintained |
| `tailwindcss` | Styling | If used; build-time only; no runtime risk |

No auth libraries, no JWT, no session middleware, no file upload libraries in v1.

---

## 6. Technology Stack

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| Framework | Next.js (App Router) | 14+ | Full-stack: SSR, React UI, Route Handlers |
| Language | TypeScript | 5.x | Strict typing throughout; `"strict": true` in tsconfig |
| UI library | React | 18.x | Component model; client-side interactivity |
| Styling | Tailwind CSS or CSS Modules | Latest | Minimal, functional styles (no UI component library) |
| Database | PostgreSQL | 14+ | Persistent note storage |
| DB client | `pg` (node-postgres) | 8.x | Parameterised SQL queries; connection pooling |
| DB client alt | Prisma | 5.x | Alternative ORM; `prisma migrate deploy` at startup |
| Runtime | Node.js LTS | 18+ | Execution environment for Next.js |
| Package manager | npm | 9+ | Dependency management and scripts |
| Config file | `next.config.js` | — | `.js` extension required on Next.js 14 (not `.ts`) |

### 6.1 File Structure

```
quicknotes2/
├── app/
│   ├── api/
│   │   ├── health/
│   │   │   └── route.ts          ← GET /health
│   │   └── notes/
│   │       ├── route.ts          ← GET + POST /api/notes
│   │       └── [id]/
│   │           └── route.ts      ← GET + PUT + DELETE /api/notes/:id
│   ├── components/
│   │   ├── ComposeBox.tsx        ← F3 compose form
│   │   ├── NoteList.tsx          ← F4 note list
│   │   └── NoteCard.tsx          ← F4/F5 card (read + edit + delete)
│   ├── layout.tsx                ← Root layout
│   └── page.tsx                  ← Root page (SSR shell)
├── lib/
│   └── db.ts                     ← Singleton pool + startup migration
├── types/
│   └── index.ts                  ← Note, CreateNoteRequest, etc.
├── next.config.js                ← Next.js config (.js extension, not .ts)
├── tsconfig.json                 ← TypeScript config (strict: true)
├── package.json
└── .env.local                    ← DATABASE_URL (local dev; git-ignored)
```

### 6.2 npm Scripts

| Script | Command | Purpose |
|--------|---------|---------|
| `npm run dev` | `next dev -p 3000 -H 0.0.0.0` | Development server on 0.0.0.0:3000 |
| `npm run build` | `next build` | Production build |
| `npm start` | `next start -p 3000 -H 0.0.0.0` | Production server on 0.0.0.0:3000 |
| `npm run lint` | `next lint` | ESLint via Next.js config |

### 6.3 Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | **Yes** | PostgreSQL connection string: `postgresql://user:password@host:port/dbname` |
| `PIVOTA_DB_MODE` | No | When set to `sidecar`, confirms direct-connect native mode (no behavioural change in v1) |

---

## 7. Integration Points

### 7.1 PostgreSQL Database

**Type:** External relational database  
**Provisioned by:** Runtime environment (not the application)  
**Connected via:** `DATABASE_URL` environment variable  

| Property | Detail |
|----------|--------|
| Protocol | PostgreSQL wire protocol (TCP) |
| Connection string format | `postgresql://user:password@host:port/dbname` |
| Client | `pg` (node-postgres) with connection pool |
| Pool model | Singleton — initialised once in `lib/db.ts`; shared across all Route Handler invocations |
| Startup migration | Idempotent DDL runs before HTTP server accepts requests |
| Sidecar mode | `PIVOTA_DB_MODE=sidecar` → same direct-connect logic; no Compose |
| Version minimum | PostgreSQL 14+ (for `gen_random_uuid()` without extension if preferred; pgcrypto used for compatibility) |

**Failure modes and handling:**

| Condition | Handling |
|-----------|----------|
| `DATABASE_URL` missing at startup | `process.exit(1)` — `"FATAL: DATABASE_URL environment variable is not set."` |
| Database unreachable at startup | `process.exit(1)` — `"FATAL: Cannot connect to database: <error>"` (password stripped) |
| Migration DDL fails | `process.exit(1)` — `"FATAL: Migration failed: <error>"` |
| Database error at request time | Route Handler catches pg error → HTTP 500 `{ "error": "Internal server error" }` |
| `notes` table already exists (restart) | `CREATE TABLE IF NOT EXISTS` is a no-op; no error |

### 7.2 Node.js Runtime

**Type:** JavaScript runtime  
**Version:** LTS (18+), compatible with Next.js 14+  

| Property | Detail |
|----------|--------|
| Process entry | `npm run dev` (dev) / `npm start` (prod) |
| Port binding | `0.0.0.0:3000` — required for sandbox/preview proxy compatibility |
| Environment | Reads `DATABASE_URL`, `PIVOTA_DB_MODE`, and standard Next.js env vars (`NODE_ENV`, etc.) |

### 7.3 Browser (Client)

**Type:** Evergreen browser (Chrome, Firefox, Safari, Edge — latest)  
**Communication:** HTTP/JSON via `fetch` API  
**Rendering:** Next.js SSR delivers initial HTML; React hydration activates interactivity client-side  

| Feature | Mechanism |
|---------|-----------|
| Note list load | `fetch('GET /api/notes')` on component mount |
| Note create | `fetch('POST /api/notes', { body: JSON.stringify({…}) })` |
| Note edit | `fetch('PUT /api/notes/:id', { body: JSON.stringify({…}) })` |
| Note delete | `fetch('DELETE /api/notes/:id')` |
| Relative timestamps | Client-side computation from `note.created_at` (ISO 8601 → human-readable) |

### 7.4 No Third-Party Integrations (v1)

QuickNotes v1 has **no external API integrations, no analytics, no CDN dependency, no email service, and no payment provider**. All data is local to the PostgreSQL instance.

---

## Appendix A: Key Architectural Decisions (ADRs)

### ADR-001: Single-Process Monolith

**Decision:** Serve UI and API from one Next.js process.  
**Rationale:** Eliminates inter-service network calls, simplifies deployment, appropriate for a single-user MVP.  
**Trade-off:** Vertical scaling only; acceptable given single-user scope.

### ADR-002: `pg` (node-postgres) as Primary DB Client

**Decision:** Use `pg` with raw parameterised SQL as the default; Prisma as documented alternative.  
**Rationale:** Thin layer gives full SQL control; no ORM abstraction overhead; predictable query behaviour.  
**Trade-off:** More verbose than an ORM; acceptable given only 1 table and 5 SQL statements.

### ADR-003: Idempotent Startup Migrations

**Decision:** Run `CREATE TABLE IF NOT EXISTS` DDL at process startup, not via a separate migration tool or Compose step.  
**Rationale:** Zero-friction deployment in sandbox environments; safe to restart at any time.  
**Trade-off:** Not suitable for destructive schema changes (column renames, drops) — those require manual intervention. Acceptable for v1 additive schema.

### ADR-004: No Authentication in v1

**Decision:** All endpoints are publicly accessible; no auth middleware.  
**Rationale:** Explicit scope decision; auth adds complexity out of proportion to a single-user tool.  
**Trade-off:** Not suitable for multi-user or internet-facing deployment without adding auth first.

### ADR-005: `next.config.js` (not `.ts`)

**Decision:** Config file uses `.js` extension.  
**Rationale:** Next.js 14 does not support `.ts` config files; using `.ts` causes startup failure.  
**Trade-off:** Config file not type-checked by TypeScript — acceptable; config is minimal.

---

*Document generated: 2026-07-04*  
*Based on: PRD-quicknotes2.md v1.0, FRD-quicknotes2.md v1.0, .planning/PROJECT.md*
