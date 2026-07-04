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
- No maximum length is enforced on `title` or `body` at the API or database layer in v1 — PostgreSQL `TEXT` is unbounded. Extremely large inputs are accepted without truncation or rejection.

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
