# Functional Requirements Document
# QuickNotes

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Version:** 1.0  
**Date:** 2026-07-04  
**Status:** Draft  
**Based On:** PRD-quicknotes2.md v1.0  

---

## Scope

This FRD specifies the functional behaviour of every feature in the QuickNotes MVP. It is intended as the single source of truth for developers: each section defines inputs, outputs, validation rules, process steps, error states, and the API/schema surface required to implement the feature without ambiguity. Out-of-scope items from the PRD (auth, rich text, search, Docker, multi-user) are not detailed here; any implementation deviation requires a PRD change first.

---

## How to Read This Document

- **Feature IDs** follow the PRD: `F0`–`F5`. Chunk files use zero-padded prefixes (`F00`–`F05`) for correct sort order.
- **Cross-references** use the form `see F02 §Process step 3` or `see Y1-api.md §Notes`.
- **API tables** in feature chunks are summaries; full request/response schemas are in `Y1-api.md`.
- **DDL** in feature chunks is a reference; canonical DDL is in `Y0-schema.md`.
- **Error tables** in feature chunks list feature-specific errors; the cross-feature catalog is in `Y2-errors.md`.
- All API endpoints return `Content-Type: application/json`.
- All error bodies have the shape `{ "error": "<human-readable message>" }`.
- TypeScript strict mode applies; no `any` in production code.

---

## Cross-Cutting Terminology

| Term | Definition |
|------|-----------|
| **Note** | A persisted record with an optional title and required body, stored in the `notes` table |
| **Body** | The required free-text content of a note (non-null, non-empty after trimming) |
| **Title** | An optional short label for a note; `null` or empty string is stored as `null`; displayed as "Untitled" in the UI |
| **DATABASE_URL** | PostgreSQL connection string injected via environment variable; required at startup |
| **PIVOTA_DB_MODE** | Optional env var signaling sidecar DB mode; when set, app connects directly and runs migrations natively |
| **Route Handler** | Next.js App Router API handler under `app/api/` |
| **Relative timestamp** | Human-readable time-since string (e.g. "2 minutes ago", "yesterday") rendered client-side |
| **Empty state** | UI condition when no notes exist; displays the message "No notes yet — add your first one above." |
| **Idempotent migration** | DDL designed to be safe to re-run on every startup (e.g. `CREATE TABLE IF NOT EXISTS`) |

---

## Master Table of Contents

| Chunk | Feature / Section |
|-------|------------------|
| [F00-scaffold-health.md](F00-scaffold-health.md) | F0: Application Scaffold & Health Endpoint |
| [F01-database-schema-connectivity.md](F01-database-schema-connectivity.md) | F1: Database Schema & Connectivity |
| [F02-notes-rest-api.md](F02-notes-rest-api.md) | F2: Notes REST API |
| [F03-compose-box-ui.md](F03-compose-box-ui.md) | F3: Single-Page UI — Compose Box |
| [F04-note-list-ui.md](F04-note-list-ui.md) | F4: Single-Page UI — Note List |
| [F05-inline-edit-delete-ui.md](F05-inline-edit-delete-ui.md) | F5: Single-Page UI — Inline Edit & Delete |
| [Y0-schema.md](Y0-schema.md) | Database Schema (full DDL) |
| [Y1-api.md](Y1-api.md) | REST API Endpoint Catalog |
| [Y2-errors.md](Y2-errors.md) | Cross-Feature Error Catalog |
| [Y3-integrations.md](Y3-integrations.md) | External Integration Points |

---
---

## F00: Application Scaffold & Health Endpoint

**Description:** This feature establishes the foundational Next.js project structure that all other features build upon. It initialises the App Router with TypeScript, starts the dev server on `0.0.0.0:3000`, and exposes a `/health` endpoint so infrastructure and operators can verify the application is live without triggering any database logic.

**Priority:** P0 — Critical. Nothing else can function without the scaffold.

**Depends On:** None.

---

### Terminology

- **App Router:** The Next.js 14+ directory-based routing system using the `app/` directory (as opposed to the legacy `pages/` directory).
- **Route Handler:** A file named `route.ts` inside `app/api/<path>/` that handles HTTP requests — the App Router equivalent of API routes.
- **`0.0.0.0` binding:** Listening on all network interfaces, required for sandbox and preview environments that proxy to the container's IP rather than `localhost`.
- **Health endpoint:** A lightweight HTTP endpoint that returns a machine-readable status with no side effects and no database dependency.

---

### Sub-features

- Next.js App Router project scaffolded with TypeScript (`tsconfig.json`, strict mode enabled)
- Dev server starts via `npm run dev` on `0.0.0.0:3000`
- `GET /health` Route Handler returning `{ "status": "ok" }` with HTTP 200
- No Docker or external runtime dependency for startup

---

### Process

1. Developer runs `npm run dev` (or `npm run build && npm start` in production).
2. Next.js reads `next.config.js` (`.js` extension required for Next.js 14 compatibility — not `.ts`).
3. Next.js binds to `0.0.0.0:3000`.
4. The Route Handler at `app/api/health/route.ts` is registered.
5. When `GET /health` is received, the handler executes with no database calls and returns `{ "status": "ok" }` immediately.
6. Response is sent with HTTP 200 and `Content-Type: application/json`.

**Note:** The health endpoint must **not** open a database connection. It is intentionally dependency-free so it can signal liveness even when the database is unavailable.

---

### Inputs

- `GET /health` — no query parameters, no request body, no authentication headers required.

---

### Outputs

- **Success:** HTTP 200, `Content-Type: application/json`, body: `{ "status": "ok" }`

---

### Validation

- No input validation required for this endpoint — it accepts any GET request.
- The endpoint must respond in under 100 ms (non-functional requirement per PRD §7).
- The `next.config.js` file must use the `.js` extension (not `.ts`) to avoid Next.js 14 config-loading issues.
- `tsconfig.json` must enable `"strict": true`.

---

### Error States

| Scenario | HTTP Status | Error Code | Notes |
|----------|-------------|------------|-------|
| Server not started / port in use | N/A — connection refused | N/A | Operator must free port 3000 before starting |
| Next.js config file uses `.ts` extension on Next 14 | N/A — startup crash | N/A | Use `.js` extension; see Risks §PRD |
| `GET /health` receives wrong method (e.g. POST) | 405 | N/A | Next.js returns 405 Method Not Allowed automatically |

---

### API Surface (this feature)

| Method | Path | Auth | Request Body | Success Response |
|--------|------|------|--------------|-----------------|
| GET | `/health` | None | None | 200 `{ "status": "ok" }` |

Full spec: see `Y1-api.md` §Health.

---

### Schema Surface (this feature)

None. The health endpoint has no database dependency.

---
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
---

## F02: Notes REST API

**Description:** This feature implements the complete CRUD (Create, Read, Update, Delete) REST API for notes as Next.js Route Handlers under `app/api/notes/`. All five endpoints return JSON, use standard HTTP status codes, and share a consistent error response format. The UI (F3–F5) calls these endpoints exclusively — there is no other data access path.

**Priority:** P0 — Critical. UI features F3, F4, F5 depend on all CRUD operations.

**Depends On:** F1 (database connection pool and `notes` table must exist).

---

### Terminology

- **Route Handler:** Next.js App Router handler file (`route.ts`) at `app/api/notes/route.ts` (collection) and `app/api/notes/[id]/route.ts` (single resource).
- **Note resource:** A single row in the `notes` table, serialised as a JSON object.
- **Collection endpoint:** `GET /api/notes` and `POST /api/notes` — operate on the full set of notes.
- **Resource endpoint:** `GET /api/notes/:id`, `PUT /api/notes/:id`, `DELETE /api/notes/:id` — operate on a single note by primary key.
- **404 Not Found:** Returned when a note with the given `id` does not exist in the database.
- **400 Bad Request:** Returned when a required field is missing, empty, or fails validation.
- **204 No Content:** Returned by `DELETE` on success; no response body.

---

### Sub-features

- `GET /api/notes` — list all notes, ordered newest first
- `POST /api/notes` — create a new note; validate required `body` field
- `GET /api/notes/:id` — retrieve a single note by ID
- `PUT /api/notes/:id` — update a note's title and/or body; bump `updated_at`
- `DELETE /api/notes/:id` — delete a note; return 204
- Consistent JSON error format for all failure cases

---

### Process

#### GET /api/notes
1. Handler receives GET request at `/api/notes`.
2. Executes `SELECT id, title, body, created_at, updated_at FROM notes ORDER BY created_at DESC`.
3. Returns HTTP 200 with JSON array (empty array `[]` if no rows exist).

#### POST /api/notes
1. Handler receives POST request with JSON body at `/api/notes`.
2. Parses request body as JSON.
3. Validates `body` field: must be present, not null, and not empty/whitespace-only after `trim()`.
4. If validation fails → return HTTP 400 `{ "error": "body is required" }`.
5. Normalises `title`: if absent, null, or empty string after `trim()`, store as `NULL`.
6. Executes `INSERT INTO notes (title, body) VALUES ($1, $2) RETURNING *`.
7. Returns HTTP 201 with the full created note JSON object.

#### GET /api/notes/:id
1. Handler receives GET request at `/api/notes/[id]`.
2. Extracts `id` from URL path params.
3. Validates `id` is a non-empty string (basic presence check; UUID format check is optional but recommended).
4. Executes `SELECT id, title, body, created_at, updated_at FROM notes WHERE id = $1`.
5. If no row returned → HTTP 404 `{ "error": "Note not found" }`.
6. Returns HTTP 200 with note JSON object.

#### PUT /api/notes/:id
1. Handler receives PUT request with JSON body at `/api/notes/[id]`.
2. Extracts `id` from URL path params.
3. Parses request body as JSON.
4. Validates `body` field: must be present, not null, and not empty/whitespace-only after `trim()`.
5. If validation fails → return HTTP 400 `{ "error": "body is required" }`.
6. Normalises `title` (same rule as POST).
7. Executes `UPDATE notes SET title = $1, body = $2, updated_at = now() WHERE id = $3 RETURNING *`.
8. If 0 rows updated → HTTP 404 `{ "error": "Note not found" }`.
9. Returns HTTP 200 with the updated note JSON object.

#### DELETE /api/notes/:id
1. Handler receives DELETE request at `/api/notes/[id]`.
2. Extracts `id` from URL path params.
3. Executes `DELETE FROM notes WHERE id = $1 RETURNING id`.
4. If 0 rows deleted → HTTP 404 `{ "error": "Note not found" }`.
5. Returns HTTP 204 with no response body.

---

### Inputs

**POST /api/notes request body:**
- `title` (string, optional): Note title. Null/missing/empty treated as null.
- `body` (string, required): Note content. Must be non-empty after trimming whitespace.

**PUT /api/notes/:id request body:**
- `title` (string, optional): Updated title. Same normalisation as POST.
- `body` (string, required): Updated body. Must be non-empty after trimming.

**Path parameter for resource endpoints:**
- `id` (string, required): Note primary key (UUID or serial integer depending on schema choice).

---

### Outputs

**Note JSON object:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "My Note Title",
  "body": "The body text of the note.",
  "created_at": "2026-07-04T12:00:00.000Z",
  "updated_at": "2026-07-04T12:05:00.000Z"
}
```
- `title` may be `null` if not set.
- `created_at` and `updated_at` are ISO 8601 UTC strings.

**GET /api/notes — array:**
```json
[
  { ...note },
  { ...note }
]
```

---

### Validation

- `body` must be a non-null string with at least one non-whitespace character (enforced on both POST and PUT).
- `title` is always optional. Empty string `""` and `null` are both normalised to `NULL` in storage.
- `id` path parameter must be present and non-empty; malformed UUIDs return 404 (no row found), not 400.
- Request body must be valid JSON; malformed JSON returns 400 `{ "error": "Invalid JSON body" }`.
- Methods not implemented on a given route return 405 (Next.js automatic behaviour).
- Server-side `body` validation is the source of truth — client-side disabled-button state (F3) is UX only, not a security control.

---

### Error States

| Scenario | HTTP Status | Body |
|----------|-------------|------|
| `body` field missing or empty (POST/PUT) | 400 | `{ "error": "body is required" }` |
| Malformed JSON request body | 400 | `{ "error": "Invalid JSON body" }` |
| Note not found by `id` (GET/PUT/DELETE) | 404 | `{ "error": "Note not found" }` |
| Database query failure | 500 | `{ "error": "Internal server error" }` |
| Wrong HTTP method on endpoint | 405 | Next.js default 405 response |

---

### API Surface (this feature)

| Method | Path | Request Body | Success Status | Success Body |
|--------|------|--------------|---------------|--------------|
| GET | `/api/notes` | None | 200 | Array of note objects |
| POST | `/api/notes` | `{ title?, body }` | 201 | Created note object |
| GET | `/api/notes/:id` | None | 200 | Single note object |
| PUT | `/api/notes/:id` | `{ title?, body }` | 200 | Updated note object |
| DELETE | `/api/notes/:id` | None | 204 | No body |

Full request/response schemas: see `Y1-api.md` §Notes.

---

### Schema Surface (this feature)

Reads and writes the `notes` table. Uses all five columns: `id`, `title`, `body`, `created_at`, `updated_at`. Full DDL: see `Y0-schema.md` §Notes.

---
---

## F03: Single-Page UI — Compose Box

**Description:** The compose box is the primary capture surface, always visible at the top of the single page. It provides an optional title input and a required body textarea so users can write and submit a new note in under 5 seconds from app open. On submit, it calls `POST /api/notes`, clears itself on success, and triggers a refresh of the note list (F4).

**Priority:** P0 — Critical. Core capture flow.

**Depends On:** F2 (`POST /api/notes` must be available). F4 (list refresh is coordinated with the note list component).

---

### Terminology

- **Compose box:** The form section at the top of the page containing the title input, body textarea, and submit button.
- **Disabled state:** The "Add Note" button rendered with `disabled` attribute and visually de-emphasised when the body is empty or whitespace-only.
- **Form clear:** Resetting both input fields to empty strings after a successful submission.
- **List refresh:** Re-fetching `GET /api/notes` and re-rendering the note list to include the newly created note.

---

### Sub-features

- Page header displaying the application title "QuickNotes"
- Optional title text input with placeholder "Title (optional)"
- Required body textarea with placeholder "Write a note…"
- "Add Note" submit button with disabled logic
- `POST /api/notes` call on submit with loading and error feedback
- Form clear on success
- Note list refresh trigger after successful submission

---

### Process

1. Page renders with compose box visible at the top.
2. Page header `<h1>` (or equivalent) displays "QuickNotes".
3. Title input is rendered as `<input type="text">` with `placeholder="Title (optional)"`, associated `<label>` ("Title"), and not required.
4. Body textarea is rendered as `<textarea>` with `placeholder="Write a note…"`, associated `<label>` ("Note"), and is functionally required.
5. "Add Note" `<button>` is rendered with `disabled` attribute when `body.trim() === ""` (empty or whitespace-only); enabled otherwise.
6. User types in the body field → button becomes enabled.
7. User clicks "Add Note" (or submits via keyboard).
8. Client trims `body` value; if empty, does not submit (button is disabled, but also guard client-side).
9. Client calls `POST /api/notes` with `{ title: titleValue.trim() || null, body: body.trim() }`.
10. While awaiting response, the button may show a loading indicator (e.g. "Saving…") and be temporarily disabled.
11. On success (HTTP 201):
    - Clear title input to `""`.
    - Clear body textarea to `""`.
    - Trigger note list refresh (re-fetch `GET /api/notes`).
12. On error (non-201 response):
    - Display an inline error message near the form (e.g. "Failed to save note. Please try again.").
    - Do not clear the form so the user can retry.

---

### Inputs

**User-facing fields:**
- `title` (text input, optional): Free-form text. Max display length unconstrained in v1.
- `body` (textarea, required): Free-form text. Must contain at least one non-whitespace character before submission is allowed.

**Submitted to API:**
- `title` (string | null): `title.trim()` or `null` if empty after trim.
- `body` (string): `body.trim()`. Non-empty (enforced before call).

---

### Outputs

- **Success path:** Note created, form cleared, note list updated.
- **Error path:** Inline error message displayed; form preserved.
- **Accessibility output:** All interactive elements reachable via Tab; labels associated via `htmlFor` / `id` pairing or `aria-label`.

---

### Validation

- Body must be non-empty after trimming before submission is allowed (client-side and API-level).
- Title is always optional; empty string submitted as `null` to the API.
- The `<button>` must carry the native HTML `disabled` attribute (not just CSS styling) so keyboard and assistive technology users also cannot activate it.
- Each form field must have an associated `<label>` element (explicit `htmlFor`/`id` association or wrapping label).
- The compose box must be rendered server-side in the initial HTML (Next.js SSR); form interactivity is handled client-side (React state).

---

### Error States

| Scenario | User-Visible Behaviour | Technical Detail |
|----------|----------------------|-----------------|
| Body is empty | "Add Note" button disabled; cannot submit | `body.trim() === ""` check |
| `POST /api/notes` returns 400 | Inline error: "Failed to save note." | Form not cleared |
| `POST /api/notes` returns 500 | Inline error: "Failed to save note. Please try again." | Form not cleared |
| Network error (fetch throws) | Inline error: "Failed to save note. Please try again." | Form not cleared |
| `POST /api/notes` succeeds but list refresh fails | Note created; list may be stale; silent retry or page reload | Non-blocking; note is persisted |

---

### API Surface (this feature)

| Method | Path | Called When | Expected Response |
|--------|------|------------|-----------------|
| POST | `/api/notes` | User clicks "Add Note" | 201 with created note |

Full schema: see `Y1-api.md` §Notes — POST.

---

### Schema Surface (this feature)

No direct database access. Reads/writes through F2 API only.

---
---

## F04: Single-Page UI — Note List

**Description:** Below the compose box, a scrollable list of all persisted notes is displayed, ordered newest first. The list is fetched from `GET /api/notes` on initial page load and refreshed after every create, edit, or delete action without a full page reload. Each note card shows the title, body, and a relative timestamp. When no notes exist, a friendly empty-state message is displayed.

**Priority:** P0 — Critical. Primary read surface for all persisted notes.

**Depends On:** F2 (`GET /api/notes` must be available), F3 (list refresh is triggered by compose box after successful create).

---

### Terminology

- **Note card:** A UI element representing a single note, displaying its title, body text, and relative timestamp.
- **Relative timestamp:** A human-readable time-since string derived from `created_at` (e.g. "just now", "2 minutes ago", "yesterday", "3 days ago"). Computed client-side.
- **Empty state:** The UI condition when `GET /api/notes` returns an empty array; displays "No notes yet — add your first one above."
- **List refresh:** Re-calling `GET /api/notes` and re-rendering the list in place without a full browser navigation or page reload.

---

### Sub-features

- Initial note list fetch from `GET /api/notes` on page load
- Rendering note cards with title, body, and relative timestamp
- "Untitled" fallback for notes with null/empty title
- Empty state message when no notes exist
- List re-renders after create (F3), edit (F5), or delete (F5) without page reload
- Notes ordered newest first (as returned by the API — no client-side re-sorting required)

---

### Process

1. Page component mounts (client-side after SSR hydration or as a client component).
2. `GET /api/notes` is called.
3. While fetching, a loading indicator may be shown (e.g. "Loading notes…") or the list area is left empty.
4. On success (HTTP 200):
   a. If the response array is empty → render empty state: `<p>No notes yet — add your first one above.</p>`.
   b. If the array has items → render one note card per item in the order returned (newest first).
5. Each note card renders:
   - **Title line:** Display `note.title` if non-null and non-empty; otherwise display `"Untitled"` (visually distinguished, e.g. italic or muted colour).
   - **Body:** Display `note.body` as plain text (no Markdown rendering in v1).
   - **Timestamp:** Display relative time derived from `note.created_at` (e.g. using a library like `date-fns` `formatDistanceToNow` or a hand-rolled helper).
   - **Edit control:** "Edit" button (see F5).
   - **Delete control:** "Delete" button (see F5).
6. When F3 (compose box) successfully creates a note, it signals the list to re-fetch → repeat from step 2.
7. When F5 (inline edit/delete) modifies or removes a note, it signals the list to re-fetch → repeat from step 2. Alternatively, for delete, the card may be removed optimistically from local state.

---

### Inputs

- `GET /api/notes` response: JSON array of note objects (see `Y1-api.md` §Notes — GET collection).
- Refresh signals: emitted by F3 after successful POST, and by F5 after successful PUT or DELETE.

---

### Outputs

- **Non-empty list:** One note card per note, in newest-first order.
- **Empty state:** Single message: "No notes yet — add your first one above."
- **Loading state:** Transient indicator while fetching (optional but recommended for UX).
- **Error state:** If `GET /api/notes` fails, display an inline error message (e.g. "Failed to load notes. Refresh to try again.").

---

### Validation

- The list must display notes in the order returned by the API (`ORDER BY created_at DESC`). No client-side re-sorting.
- `note.title` of `null`, `""`, or whitespace-only must all render as "Untitled". Do not display a blank title area.
- Relative timestamps must update if the page remains open long enough to be meaningful (a simple static render on fetch is acceptable for v1; live ticking is optional).
- The empty state message must match exactly: `"No notes yet — add your first one above."`
- List updates must not cause a full page reload; React state update and re-render is the expected mechanism.
- Accessibility: each note card must be readable by screen readers; timestamp rendered with a machine-readable `datetime` attribute on a `<time>` element (e.g. `<time dateTime={note.created_at}>2 minutes ago</time>`).

---

### Error States

| Scenario | User-Visible Behaviour | Technical Detail |
|----------|----------------------|-----------------|
| `GET /api/notes` returns empty array | Empty state message displayed | Normal condition, not an error |
| `GET /api/notes` returns 500 | "Failed to load notes. Refresh to try again." | List area shows error, compose box still usable |
| Network error on initial load | Same error message as 500 | `fetch` throws |
| List refresh fails after create/edit/delete | Previous list state retained; optional retry toast | Non-blocking; persisted data is safe |

---

### API Surface (this feature)

| Method | Path | Called When | Expected Response |
|--------|------|------------|-----------------|
| GET | `/api/notes` | Page load and after any mutation | 200 with note array |

Full schema: see `Y1-api.md` §Notes — GET collection.

---

### Schema Surface (this feature)

No direct database access. Reads through F2 API only.

---
---

## F05: Single-Page UI — Inline Edit & Delete

**Description:** Each note card in the list (F4) exposes "Edit" and "Delete" controls. The edit control transitions the card into an inline editing mode with pre-populated fields; saving calls `PUT /api/notes/:id` and updates the card in place. The delete control prompts for confirmation and then calls `DELETE /api/notes/:id`, removing the card from the list. No page navigation is required for either action.

**Priority:** P1 — High. Required for full CRUD in the UI; layered after the list (F4) is in place.

**Depends On:** F2 (`PUT /api/notes/:id` and `DELETE /api/notes/:id`), F4 (note list hosts the cards).

---

### Terminology

- **Read view:** The default rendered state of a note card — showing title, body, timestamp, and the Edit/Delete control buttons.
- **Edit mode:** The inline editing state of a note card — showing editable title input and body textarea pre-filled with current values, plus "Save" and "Cancel" buttons.
- **Confirmation prompt:** A synchronous browser dialog (`window.confirm("Are you sure?")`) or an inline confirmation widget displayed before executing a delete.
- **Optimistic removal:** Removing a note card from the list immediately on delete confirmation, before the API response, to improve perceived responsiveness. The card is restored if the API call fails.

---

### Sub-features

- "Edit" button on each note card in read view
- Inline transition to edit mode (title input + body textarea pre-populated)
- "Save" button in edit mode — calls `PUT /api/notes/:id`
- "Cancel" button in edit mode — reverts to read view without saving
- "Delete" button on each note card in read view
- Confirmation prompt before delete
- `DELETE /api/notes/:id` call on confirm; card removal from list on success
- Keyboard accessibility for all controls (Tab-navigable)

---

### Process — Edit Flow

1. Note card is in read view. "Edit" and "Delete" buttons are visible.
2. User clicks (or keyboard-activates) "Edit".
3. Card transitions to edit mode:
   - Title field: `<input type="text">` pre-filled with `note.title` (or `""` if null).
   - Body field: `<textarea>` pre-filled with `note.body`.
   - "Save" button and "Cancel" button replace the "Edit" / "Delete" buttons.
4. User modifies title and/or body.
5. "Save" button is disabled when `body.trim() === ""`.
6. User clicks "Save":
   a. Client trims values: `title = title.trim() || null`, `body = body.trim()`.
   b. Calls `PUT /api/notes/:id` with `{ title, body }`.
   c. While awaiting response: "Save" button shows loading state (e.g. "Saving…"), both buttons temporarily disabled.
   d. On success (HTTP 200):
      - Card exits edit mode and returns to read view.
      - Card displays updated title, body, and timestamp from the API response.
      - No full list re-fetch required — card state updated locally from PUT response.
   e. On error (non-200 response):
      - Card remains in edit mode.
      - Inline error displayed near the edit form (e.g. "Failed to save. Please try again.").
7. User clicks "Cancel" (at any point during edit mode):
   - Card reverts to read view with original values (no API call made).
   - Edits are discarded.

---

### Process — Delete Flow

1. Note card is in read view. "Delete" button is visible.
2. User clicks (or keyboard-activates) "Delete".
3. A confirmation prompt is shown: `"Are you sure you want to delete this note?"`
   - Implementation: `window.confirm(...)` is acceptable for v1. A custom inline widget is optional.
4. If user cancels the confirmation → no action; card remains in read view.
5. If user confirms:
   a. Card may be optimistically hidden from the list immediately (optional).
   b. Client calls `DELETE /api/notes/:id`.
   c. On success (HTTP 204):
      - Card is permanently removed from the list.
      - If the deleted note was the last one, the empty state message appears.
   d. On error (non-204 response or network failure):
      - If optimistically hidden, card is restored to the list.
      - Inline or toast error: "Failed to delete note. Please try again."

---

### Inputs

**Edit mode — user-facing fields:**
- `title` (text input, optional): Pre-filled with current `note.title` or `""`.
- `body` (textarea, required): Pre-filled with current `note.body`. Cannot be empty on save.

**Submitted to API (PUT):**
- `title` (string | null): `title.trim()` or `null` if empty.
- `body` (string): `body.trim()`. Non-empty (enforced before call).

**Delete confirmation:**
- Implicit: user accepts the browser `confirm()` dialog (or custom UI equivalent).

---

### Outputs

- **Edit success:** Updated note card in read view with new title, body, and (optionally refreshed) timestamp.
- **Edit cancel:** Card unchanged, returned to read view.
- **Delete success:** Card removed from DOM; empty state shown if no notes remain.
- **Edit/Delete error:** Inline error message; card state preserved.

---

### Validation

- Body field in edit mode must be non-empty after trimming before "Save" is enabled (same rule as F3 compose box).
- Title in edit mode is optional; normalised to `null` if empty.
- Each note card's edit/delete controls must be keyboard-reachable via Tab key and activatable via Enter/Space.
- The "Save" button must carry the native `disabled` attribute when body is empty.
- Cancelling edit must restore the exact original values (do not persist trimmed or modified intermediate state).
- Only one note card may be in edit mode at a time. If the user opens edit on one card, other cards remain in read view (or, alternatively, opening edit on a new card auto-cancels the previous edit — either is acceptable; specify behaviour in implementation).

---

### Error States

| Scenario | User-Visible Behaviour | Technical Detail |
|----------|----------------------|-----------------|
| `PUT` returns 404 (note deleted by another session) | "Note not found. It may have been deleted." | Single-user app; edge case |
| `PUT` returns 400 (body empty — server-side guard) | "body is required." | Should not occur if client validates |
| `PUT` returns 500 | "Failed to save. Please try again." | Card stays in edit mode |
| `DELETE` returns 404 | "Note not found." Remove card anyway. | Card already gone from DB |
| `DELETE` returns 500 | "Failed to delete. Please try again." | Restore card if optimistically removed |
| Network error on PUT/DELETE | Same as 500 behaviour above | `fetch` throws |
| User cancels delete confirmation | No action, card remains | `confirm()` returns `false` |
| Body empty in edit mode | "Save" button disabled | Client-side only guard |

---

### API Surface (this feature)

| Method | Path | Called When | Expected Response |
|--------|------|------------|-----------------|
| PUT | `/api/notes/:id` | User clicks "Save" in edit mode | 200 with updated note |
| DELETE | `/api/notes/:id` | User confirms delete | 204 No Content |

Full schemas: see `Y1-api.md` §Notes — PUT and DELETE.

---

### Schema Surface (this feature)

No direct database access. Reads/writes through F2 API only.

---
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
---

## REST API Endpoint Catalog

**Implemented by:** F0 (Health), F2 (Notes CRUD)  
**Consumed by:** F3 (Compose Box), F4 (Note List), F5 (Inline Edit & Delete)  
**Base URL:** `http://localhost:3000` (dev)  
**Content-Type:** All requests and responses use `application/json`  
**Auth:** None (single-user, no auth in v1)  

---

### §Health

#### GET /health

Returns a liveness signal. No database dependency. Must respond in under 100 ms.

**Request:**
- Method: `GET`
- Path: `/health`
- Headers: none required
- Body: none

**Response — 200 OK:**
```json
{
  "status": "ok"
}
```

**Error responses:** None expected. If the server is down, the connection is refused.

---

### §Notes

#### Note Object Schema

All note endpoints return note objects in this shape:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Optional Title",
  "body": "The body text of the note.",
  "created_at": "2026-07-04T12:00:00.000Z",
  "updated_at": "2026-07-04T12:05:00.000Z"
}
```

| Field | Type | Nullable | Description |
|-------|------|----------|-------------|
| `id` | `string` (UUID) | No | Primary key |
| `title` | `string` \| `null` | Yes | Note title; `null` when not set |
| `body` | `string` | No | Note body text |
| `created_at` | `string` (ISO 8601 UTC) | No | Creation timestamp |
| `updated_at` | `string` (ISO 8601 UTC) | No | Last update timestamp |

---

#### GET /api/notes

Retrieves all notes ordered by `created_at` descending (newest first).

**Request:**
- Method: `GET`
- Path: `/api/notes`
- Headers: none required
- Query params: none
- Body: none

**Response — 200 OK:**
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
Returns an empty array `[]` when no notes exist.

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 500 | `{ "error": "Internal server error" }` | Database query failure |

---

#### POST /api/notes

Creates a new note. Returns the created note with HTTP 201.

**Request:**
- Method: `POST`
- Path: `/api/notes`
- Headers: `Content-Type: application/json`
- Body:

```json
{
  "title": "Optional Title",
  "body": "Required body text."
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `title` | `string` \| `null` | No | Stored as `NULL` if absent, `null`, or empty after trim |
| `body` | `string` | Yes | Must be non-empty after trim; returns 400 otherwise |

**Response — 201 Created:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Optional Title",
  "body": "Required body text.",
  "created_at": "2026-07-04T12:00:00.000Z",
  "updated_at": "2026-07-04T12:00:00.000Z"
}
```

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 400 | `{ "error": "body is required" }` | `body` missing, null, or empty/whitespace |
| 400 | `{ "error": "Invalid JSON body" }` | Request body is not valid JSON |
| 500 | `{ "error": "Internal server error" }` | Database insert failure |

---

#### GET /api/notes/:id

Retrieves a single note by its UUID.

**Request:**
- Method: `GET`
- Path: `/api/notes/:id`
- Path param `id`: UUID string (the note's primary key)
- Headers: none required
- Body: none

**Response — 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "My Title",
  "body": "Note body text.",
  "created_at": "2026-07-04T12:00:00.000Z",
  "updated_at": "2026-07-04T12:00:00.000Z"
}
```

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 404 | `{ "error": "Note not found" }` | No note with given `id` exists |
| 500 | `{ "error": "Internal server error" }` | Database query failure |

---

#### PUT /api/notes/:id

Updates an existing note's title and/or body. Bumps `updated_at`. Returns the updated note.

**Request:**
- Method: `PUT`
- Path: `/api/notes/:id`
- Path param `id`: UUID string
- Headers: `Content-Type: application/json`
- Body:

```json
{
  "title": "Updated Title",
  "body": "Updated body text."
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `title` | `string` \| `null` | No | Same normalisation as POST |
| `body` | `string` | Yes | Must be non-empty after trim; returns 400 otherwise |

**Response — 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Updated Title",
  "body": "Updated body text.",
  "created_at": "2026-07-04T12:00:00.000Z",
  "updated_at": "2026-07-04T12:10:00.000Z"
}
```

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 400 | `{ "error": "body is required" }` | `body` missing, null, or empty/whitespace |
| 400 | `{ "error": "Invalid JSON body" }` | Request body is not valid JSON |
| 404 | `{ "error": "Note not found" }` | No note with given `id` exists |
| 500 | `{ "error": "Internal server error" }` | Database update failure |

---

#### DELETE /api/notes/:id

Deletes a note by its UUID. Returns 204 No Content on success.

**Request:**
- Method: `DELETE`
- Path: `/api/notes/:id`
- Path param `id`: UUID string
- Headers: none required
- Body: none

**Response — 204 No Content:**
- No response body.

**Error responses:**

| Status | Body | Condition |
|--------|------|-----------|
| 404 | `{ "error": "Note not found" }` | No note with given `id` exists |
| 500 | `{ "error": "Internal server error" }` | Database delete failure |

---

### §Endpoint Summary

| Method | Path | Feature | Success Status |
|--------|------|---------|---------------|
| GET | `/health` | F0 | 200 |
| GET | `/api/notes` | F2 | 200 |
| POST | `/api/notes` | F2 | 201 |
| GET | `/api/notes/:id` | F2 | 200 |
| PUT | `/api/notes/:id` | F2 | 200 |
| DELETE | `/api/notes/:id` | F2 | 204 |

---
---

## Cross-Feature Error Catalog

This catalog lists all error conditions across QuickNotes features, their HTTP status codes, error codes, user/client-facing messages, and handling guidance.

---

### §Startup Errors (F1 — Database Connectivity)

These errors occur at process startup and are terminal (the process exits).

| Condition | Exit Code | Log Message | Handling |
|-----------|-----------|-------------|----------|
| `DATABASE_URL` not set | 1 | `"FATAL: DATABASE_URL environment variable is not set."` | Set `DATABASE_URL` before starting |
| Database unreachable | 1 | `"FATAL: Cannot connect to database: <pg error>"` | Verify PostgreSQL is running and URL is correct |
| Migration DDL fails | 1 | `"FATAL: Migration failed: <pg error>"` | Check DB user permissions; inspect pg error |

---

### §API Errors (F2 — Notes REST API)

These errors are returned as HTTP responses during normal operation.

| HTTP Status | Error Code (body `error` field) | Trigger Condition | Feature |
|-------------|--------------------------------|------------------|---------|
| 400 | `"body is required"` | `POST /api/notes` or `PUT /api/notes/:id` with missing/empty/whitespace body | F2 |
| 400 | `"Invalid JSON body"` | Request body cannot be parsed as JSON (POST or PUT) | F2 |
| 404 | `"Note not found"` | `GET`, `PUT`, or `DELETE /api/notes/:id` — no row matches `id` | F2 |
| 405 | (Next.js default) | HTTP method not implemented for a given route | F2 |
| 500 | `"Internal server error"` | Unhandled database or server error during query execution | F2 |

**Error response format (all 4xx/5xx):**
```json
{ "error": "<message string>" }
```

---

### §UI Errors (F3, F4, F5 — User Interface)

These errors are surfaced to the user as inline messages within the UI. They do not cause page reloads or navigation.

| Scenario | User-Visible Message | Feature | Recovery |
|----------|---------------------|---------|----------|
| POST /api/notes fails | "Failed to save note. Please try again." | F3 | Form preserved; user retries |
| GET /api/notes fails on load | "Failed to load notes. Refresh to try again." | F4 | Manual page refresh |
| GET /api/notes fails on refresh | Previous list state shown; optional toast | F4 | Automatic retry or manual refresh |
| PUT /api/notes/:id fails | "Failed to save. Please try again." | F5 | Card stays in edit mode; user retries |
| DELETE /api/notes/:id fails | "Failed to delete note. Please try again." | F5 | Card restored (if optimistically removed) |
| Note no longer exists on PUT/DELETE (404) | "Note not found. It may have been deleted." | F5 | Remove card from list; rare in single-user app |

---

### §Validation Errors

Client-side validation prevents these from reaching the API, but the API enforces them server-side as a second line of defence.

| Rule | Client Behaviour | API Response if Bypassed |
|------|-----------------|------------------------|
| `body` is required (non-empty after trim) | "Add Note" / "Save" button disabled | 400 `{ "error": "body is required" }` |
| `title` is optional | No client restriction | Stored as `NULL` |
| `id` must match an existing note | Not validated client-side | 404 `{ "error": "Note not found" }` |
| Request body must be valid JSON | Serialised by `fetch` / `JSON.stringify` | 400 `{ "error": "Invalid JSON body" }` |

---

### §Non-Functional Error Conditions

| Condition | Expected Behaviour | Notes |
|-----------|-------------------|-------|
| Port 3000 already in use | Process fails to start; OS error | Operator must free port |
| `next.config.js` uses `.ts` extension on Next.js 14 | Build/startup failure | Use `.js` extension |
| `DATABASE_URL` set but credentials wrong | Startup fatal (connect error) | Verify pg credentials |

---
---

## External Integration Points

QuickNotes v1 has minimal external dependencies by design. This section catalogs all external systems the application touches, their contracts, and failure modes.

---

### §PostgreSQL Database

**Type:** Relational database  
**Used by:** F1 (schema & connectivity), F2 (all CRUD operations)  
**Connection:** Via `DATABASE_URL` environment variable  

| Property | Value |
|----------|-------|
| Protocol | PostgreSQL wire protocol (TCP) |
| Connection string format | `postgresql://user:password@host:port/dbname` |
| Client library | `pg` (node-postgres) or Prisma |
| Connection model | Singleton connection pool (shared across all Route Handler invocations) |
| Startup behaviour | Must be reachable before app serves requests; fail-fast if not |
| Migration strategy | Idempotent DDL (`CREATE TABLE IF NOT EXISTS`) run at startup; OR `prisma migrate deploy` / `prisma db push` |
| Sidecar mode | When `PIVOTA_DB_MODE=sidecar` is set, connect directly — same connection logic as normal mode in v1 |

**Failure modes:**
- Unreachable at startup → process exits with code 1 (see `Y2-errors.md` §Startup Errors).
- Unreachable at request time → Route Handler catches pg error and returns HTTP 500.

**Security note:** `DATABASE_URL` contains credentials. Never log the full URL; strip the password when logging connection errors.

---

### §Node.js Runtime

**Type:** JavaScript/TypeScript runtime  
**Version:** Node.js LTS (compatible with Next.js 14+)  
**Used by:** All features  

| Property | Value |
|----------|-------|
| Entry point | `npm run dev` (development) / `npm start` (production) |
| Port binding | `0.0.0.0:3000` |
| Environment | Reads `DATABASE_URL`, `PIVOTA_DB_MODE`, and standard Next.js env vars |

**No Docker dependency.** The app must run with `node` / `npm` directly. Docker or docker-compose must not be required at runtime.

---

### §Browser (Client-Side)

**Type:** Web browser (evergreen)  
**Supported browsers:** Chrome, Firefox, Safari, Edge (latest versions)  
**Used by:** F3, F4, F5 (UI features)  

| Property | Value |
|----------|-------|
| Protocol | HTTP (or HTTPS in production) |
| API communication | `fetch` API (native browser, no external HTTP client library required) |
| Rendering | Next.js SSR for initial HTML; React client-side hydration for interactivity |
| No native mobile app | Web only in v1 |

**Browser APIs used:**
- `fetch` — for all API calls to `/api/notes` and `/health`
- `window.confirm` — for delete confirmation prompt (v1; may be replaced with custom widget)
- `Date` — for relative timestamp computation

---

### §Out-of-Scope Integrations (v1)

The following external integrations are explicitly **not** implemented in v1:

| Integration | Status | Notes |
|-------------|--------|-------|
| Authentication / Identity Provider (e.g. Auth0, Azure AD) | Out of scope | Single-user, no auth |
| Email / notification service | Out of scope | No notifications in v1 |
| File storage (S3, local disk) | Out of scope | No file attachments in v1 |
| WebSocket / real-time sync | Out of scope | Single-user; no cross-tab sync |
| Docker / docker-compose | Out of scope — intentionally excluded | Runtime provisions DB natively |
| CDN / edge caching | Out of scope | Single Next.js process serves everything |

---
