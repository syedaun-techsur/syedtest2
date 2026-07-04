# Flow-04: Developer Health Check & API Verification

**User Story:** US-0.1, US-0.2, US-1.1–1.3, US-2.1–2.5  
**Persona:** PER-02 Jordan Kim (Developer)  
**Journey:** JRN-02.1 — First-Run Deployment, JRN-02.2 — REST API Contract Validation  
**Trigger:** Jordan clones the repo and runs `npm run dev`  
**Exit (success):** All CRUD endpoints pass contract checks  
**Note:** This flow is primarily terminal/API-driven, not a UI flow. It is documented for completeness and to surface the minimal "developer UI" surfaces.

---

## Flow Diagram — Startup & Health

```
[Jordan: git clone → npm install]
         │
         ▼
[Jordan: export DATABASE_URL=... (or .env.local)]
         │
         ▼
[Jordan: npm run dev]
         │
         ▼
[Server starts — startup sequence:]
[1. Read DATABASE_URL from environment]
         │
         ├── DATABASE_URL missing ──▶ [Process exits code 1]
         │                             Log: "FATAL: DATABASE_URL environment
         │                              variable is not set."
         │                             [Jordan fixes env and restarts]
         │
         ├── DATABASE_URL present ──▶ [Connect to PostgreSQL pool]
         │                                      │
         │              DB unreachable ──────────┤
         │                                      │
         │                            [Process exits code 1]
         │                             Log: "FATAL: Cannot connect to database: <pg>"
         │
         └── DB connected ──────────▶ [Run idempotent migrations]
                                       CREATE TABLE IF NOT EXISTS notes ...
                                       CREATE INDEX IF NOT EXISTS ...
                                                │
                                   Migration fails ──▶ [Exit code 1]
                                   (rare: permissions)  Log: "FATAL: Migration failed: <pg>"
                                                │
                                   Migration OK ───▶ [Server ready on 0.0.0.0:3000]
                                                │
                                                ▼
                               [Jordan: curl http://localhost:3000/health]
                                                │
                                                ▼
                               [GET /health → 200 { "status": "ok" }]
                                         (< 100 ms, no DB call)
                                                │
                                                ▼
                                            [EXIT — Health verified]
```

---

## Flow Diagram — API Contract Verification

```
[App running, health confirmed]
         │
         ▼
[Jordan: POST /api/notes { "body": "Test note" }]
         │  Expected: 201 + full note JSON (id, title, body, created_at, updated_at)
         │
         ▼
[Jordan: GET /api/notes]
         │  Expected: 200 + JSON array, newest note at index 0
         │
         ▼
[Jordan: GET /api/notes/:id]
         │  Expected: 200 + note JSON matching list item shape
         │
         ▼
[Jordan: PUT /api/notes/:id { "body": "Updated" }]
         │  Expected: 200 + note JSON, updated_at bumped
         │
         ▼
[Jordan: DELETE /api/notes/:id]
         │  Expected: 204 No Content
         │
         ▼
[Jordan: Error cases]
  POST with empty body     → 400 { "error": "body is required" }
  GET /api/notes/unknown   → 404 { "error": "Note not found" }
  PUT /api/notes/unknown   → 404 { "error": "Note not found" }
  DELETE /api/notes/unknown→ 404 { "error": "Note not found" }
         │
         ▼
[All checks pass → API contract verified]
[EXIT — Success]
```

---

## Error / Startup States Visible to Developer

| Condition | Output | Requirement |
|---|---|---|
| `DATABASE_URL` not set | Terminal: `"FATAL: DATABASE_URL environment variable is not set."`, exit 1 | US-1.3 |
| DB unreachable at startup | Terminal: `"FATAL: Cannot connect to database: <pg error>"`, exit 1 | US-1.3 |
| Migration DDL fails | Terminal: `"FATAL: Migration failed: <pg error>"`, exit 1 | US-1.3 |
| Restart with existing table | No error — `IF NOT EXISTS` is a no-op, server starts normally | US-1.2 |
| `GET /health` — correct method | HTTP 200 `{ "status": "ok" }`, `Content-Type: application/json` | US-0.2 |
| `POST /health` — wrong method | HTTP 405 | US-0.2 |

---

## Security Note
The full `DATABASE_URL` (including credentials) must **never** appear in logs. The password segment must be stripped from any error message. (US-1.3)

---

## No UI Surfaces Required
This flow has no UI screen designs. All feedback is via:
- Terminal log output (startup errors)
- HTTP responses (health endpoint, API)
- PostgreSQL schema (verifiable via `psql`)
