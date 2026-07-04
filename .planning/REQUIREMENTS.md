# Requirements: QuickNotes

**Defined:** 2026-07-04
**Core Value:** A user can capture a note in under 5 seconds and trust it will be there when they return — fast capture, reliable persistence.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### F0 — Scaffold & Health

- [ ] **F0-01**: User can start the application via `npm run dev` on port 3000 bound to `0.0.0.0`
- [ ] **F0-02**: User can verify app health via `GET /health` returning HTTP 200 with JSON `{ "status": "ok" }`
- [ ] **F0-03**: App is scaffolded as Next.js App Router with TypeScript using a compatible config file (`next.config.mjs` or `next.config.js`)

### F1 — Database Schema & Connectivity

- [ ] **F1-01**: App connects to PostgreSQL using the `DATABASE_URL` environment variable at startup (no Docker dependency)
- [ ] **F1-02**: App fails fast at startup with a clear error if `DATABASE_URL` is missing or DB is unreachable
- [ ] **F1-03**: `notes` table is created/migrated natively at startup (idempotent) with columns: `id` (uuid/serial PK), `title` (text, nullable), `body` (text, not null), `created_at` (timestamptz, default now()), `updated_at` (timestamptz, default now(), bumped on edit)

### F2 — Notes REST API

- [ ] **F2-01**: `GET /api/notes` returns all notes ordered by `created_at` descending (newest first)
- [ ] **F2-02**: `POST /api/notes` creates a note with `{ title?, body }`; returns 400 if `body` is empty; returns the created note with 201
- [ ] **F2-03**: `GET /api/notes/:id` returns a single note; returns 404 if not found
- [ ] **F2-04**: `PUT /api/notes/:id` updates a note's `title` and/or `body`, bumps `updated_at`; returns 404 if not found
- [ ] **F2-05**: `DELETE /api/notes/:id` deletes a note; returns 404 if not found (or silently succeeds if already deleted)

### F3 — UI Compose Box

- [ ] **F3-01**: User sees a compose area with an optional title input and a required body textarea
- [ ] **F3-02**: "Add Note" button is disabled when the body textarea is empty
- [ ] **F3-03**: Submitting a valid note calls `POST /api/notes`, clears the form, and shows the new note at the top of the list

### F4 — UI Note List & Empty State

- [ ] **F4-01**: User sees a list of notes below the compose box, ordered newest first, each showing title (or "Untitled"), body, and relative timestamp
- [ ] **F4-02**: User sees "No notes yet — add your first one above." when no notes exist
- [ ] **F4-03**: Note list loads within 500ms of page render

### F5 — UI Inline Edit & Delete

- [ ] **F5-01**: User can click Edit on a note to enter inline edit mode, modify title/body, and save
- [ ] **F5-02**: User can cancel an inline edit, reverting to the original values
- [ ] **F5-03**: User can delete a note with a confirmation prompt; note is removed from the list on confirmation
- [ ] **F5-04**: Only one note can be in edit mode at a time (opening a second auto-cancels the first)

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Enhanced Features

- **V2-01**: Rich-text / markdown formatting for note bodies
- **V2-02**: Tags and categorization for notes
- **V2-03**: Full-text search across notes
- **V2-04**: File/image attachments
- **V2-05**: Real-time sync across tabs (WebSocket/SSE)

### Multi-User

- **V2-06**: User authentication (email/password)
- **V2-07**: Per-user note isolation

## Out of Scope

| Feature | Reason |
|---------|--------|
| Authentication / multi-user accounts | Single implicit user; auth adds significant complexity not needed for v1 |
| Rich-text, tags, search, file attachments | Future work; out of scope per spec |
| Real-time sync across tabs | Not needed for single-user experience |
| Docker / docker-compose for DB | Runtime provisions database; app must work without Docker |
| OAuth / SSO | No auth at all in v1 |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| F0-01 | Phase 1 | Pending |
| F0-02 | Phase 1 | Pending |
| F0-03 | Phase 1 | Pending |
| F1-01 | Phase 2 | Pending |
| F1-02 | Phase 2 | Pending |
| F1-03 | Phase 2 | Pending |
| F2-01 | Phase 3 | Pending |
| F2-02 | Phase 3 | Pending |
| F2-03 | Phase 3 | Pending |
| F2-04 | Phase 3 | Pending |
| F2-05 | Phase 3 | Pending |
| F3-01 | Phase 4 | Pending |
| F3-02 | Phase 4 | Pending |
| F3-03 | Phase 4 | Pending |
| F4-01 | Phase 4 | Pending |
| F4-02 | Phase 4 | Pending |
| F4-03 | Phase 4 | Pending |
| F5-01 | Phase 5 | Pending |
| F5-02 | Phase 5 | Pending |
| F5-03 | Phase 5 | Pending |
| F5-04 | Phase 5 | Pending |

**Coverage:**
- v1 requirements: 21 total
- Mapped to phases: 21
- Unmapped: 0 ✓

---
*Requirements defined: 2026-07-04*
*Last updated: 2026-07-04 after initial definition*
