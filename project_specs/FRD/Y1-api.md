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
