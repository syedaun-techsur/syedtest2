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
