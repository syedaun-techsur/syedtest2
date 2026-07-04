# Requirements Traceability Matrix
# QuickNotes

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Version:** 1.0  
**Date:** 2026-07-04  
**Status:** Draft  
**Based On:** PRD-quicknotes2.md v1.0, FRD-quicknotes2.md v1.0, TechArch-quicknotes2.md v1.0, UserStories-quicknotes2.md v1.0  

---

## 1. Overview

This Requirements Traceability Matrix (RTM) provides bidirectional traceability between all QuickNotes specification documents. It ensures that every product requirement defined in the PRD is decomposed into functional requirements in the FRD, implemented through a verifiable technical architecture specification, exercised by at least one user story, and covered by a corresponding set of test cases. The RTM is the authoritative linkage document throughout the QuickNotes v1 development lifecycle.

Traceability operates across four levels. At the highest level, PRD features (F0–F5) capture the product-level intent: what the system must do and why it matters to the user. These features decompose into FRD sub-features, which define precise inputs, outputs, validation rules, process steps, and error states without ambiguity. The Technical Architecture Document maps FRD requirements onto implementation specifications — naming the exact files, components, SQL statements, and architectural decisions (SPEC and ADR identifiers) that realise each requirement. Finally, User Stories (US-0.1 through US-5.5) provide the acceptance-criteria layer that validates implementation against user-observable behaviour.

This document is structured so that any stakeholder — product owner, developer, QA engineer, or auditor — can trace a requirement forward from business intent to implementation detail, or backward from a test failure to the originating PRD feature. All IDs referenced herein are extracted directly from the source specification documents; no placeholder identifiers are used.

---

## 2. Requirements Summary

### 2.1 PRD Features

- **F0 — Application Scaffold & Health Endpoint** (P0 — Critical): Next.js App Router project with TypeScript; `npm run dev` on `0.0.0.0:3000`; `GET /health` returns 200 `{ "status": "ok" }`; no Docker dependency.
- **F1 — Database Schema & Connectivity** (P0 — Critical): PostgreSQL connection via `DATABASE_URL`; idempotent `notes` table migration at startup; fail-fast on missing or invalid config; sidecar mode via `PIVOTA_DB_MODE`.
- **F2 — Notes REST API** (P0 — Critical): Full CRUD REST API — `GET /api/notes`, `POST /api/notes`, `GET /api/notes/:id`, `PUT /api/notes/:id`, `DELETE /api/notes/:id` — as Next.js Route Handlers with standard HTTP status codes and consistent JSON error format.
- **F3 — Single-Page UI — Compose Box** (P0 — Critical): Always-visible compose form at the top of the page; optional title input; required body textarea; "Add Note" button with disabled logic; `POST /api/notes` on submit; form clear and list refresh on success; inline error on failure.
- **F4 — Single-Page UI — Note List** (P0 — Critical): Live list of all notes below the compose box, newest first; each card shows title (or "Untitled"), body, and relative timestamp; empty-state message; updates without page reload after any mutation.
- **F5 — Single-Page UI — Inline Edit & Delete** (P1 — High): Inline edit mode per card with pre-populated fields; `PUT /api/notes/:id` on save; cancel to revert; delete with confirmation prompt; `DELETE /api/notes/:id` on confirm; keyboard accessible.

### 2.2 FRD Sub-Features by Feature

- **F0**: Next.js App Router scaffold; TypeScript strict mode; `0.0.0.0:3000` binding; `GET /health` Route Handler (no DB dependency); `next.config.js` extension constraint.
- **F1**: `DATABASE_URL` validation at startup; singleton connection pool; idempotent `notes` DDL migration; `PIVOTA_DB_MODE` sidecar support; fail-fast exit with descriptive log messages.
- **F2**: `GET /api/notes` (list, newest first); `POST /api/notes` (create with body validation); `GET /api/notes/:id` (single note); `PUT /api/notes/:id` (update with `updated_at` bump); `DELETE /api/notes/:id` (204 on success); consistent JSON error format.
- **F3**: Page header "QuickNotes"; title `<input>` with label; body `<textarea>` with label; "Add Note" button with native `disabled`; `POST /api/notes` call; form clear on 201; list refresh trigger; inline error on non-201.
- **F4**: Initial `GET /api/notes` fetch on page load; note card rendering (title/body/timestamp); "Untitled" fallback; empty state message; list refresh after create/edit/delete; `<time dateTime>` element for timestamps.
- **F5**: Edit button → inline edit mode with pre-populated fields; Save (`PUT`) and Cancel buttons; single-card-edit-at-a-time rule; Delete button → confirmation prompt; `DELETE /api/notes/:id` on confirm; optimistic removal with rollback; keyboard accessibility for all controls.

### 2.3 TechArch Specifications

- **SPEC-001** — DB Layer singleton (`lib/db.ts`): startup migration, fail-fast exit, pool export.
- **SPEC-002** — Health Route Handler (`app/api/health/route.ts`): zero DB dependency, < 100 ms SLA.
- **SPEC-003** — Notes Collection Handler (`app/api/notes/route.ts`): `GET` (SELECT ORDER BY created_at DESC) and `POST` (INSERT RETURNING *).
- **SPEC-004** — Note Resource Handler (`app/api/notes/[id]/route.ts`): `GET`, `PUT` (UPDATE SET updated_at = now()), `DELETE` (RETURNING id).
- **SPEC-005** — `ComposeBox` component (`app/components/ComposeBox.tsx`): local state, disabled logic, POST call, onNoteCreated prop.
- **SPEC-006** — `NoteList` component (`app/components/NoteList.tsx`): fetchNotes, refresh, loading/error/empty/list render.
- **SPEC-007** — `NoteCard` component (`app/components/NoteCard.tsx`): read view, edit mode, PUT save, DELETE confirm, onRefresh prop.
- **SPEC-008** — Data model (`notes` table): UUID PK, TEXT title (nullable), TEXT body (NOT NULL), TIMESTAMPTZ created_at/updated_at; idempotent DDL; `idx_notes_created_at` index.
- **SPEC-009** — TypeScript shared types (`types/index.ts`): `Note`, `CreateNoteRequest`, `UpdateNoteRequest`, `ApiError`, `HealthResponse` interfaces; strict mode.
- **ADR-001** — Single-process monolith: one Next.js process serves UI and API.
- **ADR-002** — `pg` (node-postgres) as primary DB client with parameterised SQL.
- **ADR-003** — Idempotent startup migrations (no Compose, no separate migration runner).
- **ADR-004** — No authentication in v1.
- **ADR-005** — `next.config.js` uses `.js` extension (not `.ts`).

### 2.4 User Stories

- **22 stories** across 6 epics: 17 P0 (Critical) and 5 P1 (High).
- Epics map 1-to-1 with PRD features F0–F5.
- Personas: PER-01 Alex Rivera (knowledge worker, end-user stories) and PER-02 Jordan Kim (developer/technical evaluator, API and infrastructure stories).

### 2.5 Non-Functional Requirements

- Performance: note creation round-trip < 500 ms; page initial load < 2 s; `GET /health` < 100 ms.
- Reliability: zero data loss across server restarts; fail-fast on missing `DATABASE_URL`.
- Portability: no Docker runtime dependency; `0.0.0.0:3000` binding.
- Correctness: body validation returns 400; missing resource returns 404.
- Maintainability: TypeScript strict mode; no `any` in production code; single deployable unit.
- Security: parameterised SQL (no injection surface); credentials never logged; React default XSS escaping.

---

## 3. Traceability Matrix

### 3.1 PRD Feature → FRD Sub-Feature → TechArch Spec → User Story

| PRD Feature | FRD Sub-Feature | TechArch Spec | User Story |
|-------------|-----------------|---------------|------------|
| F0: Application Scaffold & Health Endpoint | Next.js App Router scaffold; TypeScript strict mode; `tsconfig.json` `"strict": true` | ADR-001 (single-process monolith); ADR-005 (`next.config.js` `.js` extension); SPEC-009 (TypeScript types, strict mode) | US-0.1 (Start the Application) |
| F0: Application Scaffold & Health Endpoint | Dev server starts on `0.0.0.0:3000` via `npm run dev` | ADR-001; npm script `next dev -p 3000 -H 0.0.0.0` | US-0.1 (Start the Application) |
| F0: Application Scaffold & Health Endpoint | `GET /health` Route Handler — no DB dependency; 200 `{ "status": "ok" }`; < 100 ms | SPEC-002 (Health Route Handler `app/api/health/route.ts`) | US-0.2 (Verify Application Health) |
| F1: Database Schema & Connectivity | `DATABASE_URL` validation at module init; fail-fast exit code 1 | SPEC-001 (DB Layer `lib/db.ts`); ADR-003 (idempotent startup migrations) | US-1.1 (Connect to the Database at Startup); US-1.3 (Fail Fast on Missing or Invalid Database Configuration) |
| F1: Database Schema & Connectivity | Singleton `pg.Pool` established from `DATABASE_URL`; reused across Route Handlers | SPEC-001 (`lib/db.ts` singleton pool export); ADR-002 (`pg` client with parameterised SQL) | US-1.1 (Connect to the Database at Startup) |
| F1: Database Schema & Connectivity | Idempotent `notes` table DDL (`CREATE TABLE IF NOT EXISTS`); `pgcrypto` extension; `idx_notes_created_at` index | SPEC-008 (notes table DDL + indexes); ADR-003 (idempotent startup migrations) | US-1.2 (Migrate the Database Schema on Startup) |
| F1: Database Schema & Connectivity | `PIVOTA_DB_MODE` sidecar support; direct-connect native mode | SPEC-001 (`lib/db.ts`); environment variable `PIVOTA_DB_MODE` | US-1.1 (Connect to the Database at Startup) |
| F1: Database Schema & Connectivity | Fail-fast: DB unreachable → exit 1 `"FATAL: Cannot connect to database: <pg error>"`; migration failure → exit 1 | SPEC-001 (`lib/db.ts` fail-fast logic) | US-1.3 (Fail Fast on Missing or Invalid Database Configuration) |
| F2: Notes REST API | `GET /api/notes` — SELECT ORDER BY created_at DESC; 200 + array; `[]` when empty | SPEC-003 (Notes Collection Handler `app/api/notes/route.ts`) | US-2.1 (List All Notes) |
| F2: Notes REST API | `POST /api/notes` — body validation; INSERT RETURNING *; 201 + created note; 400 on empty body; 400 on bad JSON | SPEC-003 (Notes Collection Handler); SPEC-008 (notes table schema) | US-2.2 (Create a Note via API) |
| F2: Notes REST API | `GET /api/notes/:id` — SELECT WHERE id = $1; 200 + note; 404 if not found | SPEC-004 (Note Resource Handler `app/api/notes/[id]/route.ts`) | US-2.3 (Retrieve a Single Note via API) |
| F2: Notes REST API | `PUT /api/notes/:id` — body validation; UPDATE SET updated_at = now() RETURNING *; 200 + updated note; 400/404 errors | SPEC-004 (Note Resource Handler); SPEC-008 (notes table `updated_at`) | US-2.4 (Update a Note via API) |
| F2: Notes REST API | `DELETE /api/notes/:id` — DELETE RETURNING id; 204 no body; 404 if not found | SPEC-004 (Note Resource Handler) | US-2.5 (Delete a Note via API) |
| F2: Notes REST API | Consistent JSON error format `{ "error": "<message>" }` for all 4xx/5xx; parameterised SQL (no injection) | SPEC-003; SPEC-004; ADR-002 (pg parameterised SQL) | US-2.1, US-2.2, US-2.3, US-2.4, US-2.5 |
| F3: Single-Page UI — Compose Box | Page header "QuickNotes" `<h1>`; compose box always visible at top on load | SPEC-005 (`ComposeBox.tsx`); `app/page.tsx` SSR shell | US-3.1 (Compose and Submit a New Note) |
| F3: Single-Page UI — Compose Box | Title `<input type="text">` with label + placeholder "Title (optional)"; body `<textarea>` with label + placeholder "Write a note…" | SPEC-005 (`ComposeBox.tsx` local state: title, body) | US-3.1 (Compose and Submit a New Note); US-3.4 (Compose Box Accessibility) |
| F3: Single-Page UI — Compose Box | "Add Note" `<button disabled={body.trim() === '' \|\| isSubmitting}>`; native HTML `disabled` attribute | SPEC-005 (`ComposeBox.tsx` disabled logic) | US-3.2 (Prevent Empty Note Submission) |
| F3: Single-Page UI — Compose Box | On submit: `POST /api/notes`; on 201: clear form, call `onNoteCreated()` refresh prop | SPEC-005 (`ComposeBox.tsx` onNoteCreated prop → NoteList refresh) | US-3.1 (Compose and Submit a New Note); US-3.3 (Handle Compose Box Errors Gracefully) |
| F3: Single-Page UI — Compose Box | On non-201 / network error: inline error "Failed to save note. Please try again."; form preserved | SPEC-005 (`ComposeBox.tsx` error state) | US-3.3 (Handle Compose Box Errors Gracefully) |
| F3: Single-Page UI — Compose Box | All controls Tab-navigable; labels via `htmlFor`/`id`; compose box in initial SSR HTML | SPEC-005 (`ComposeBox.tsx`); `app/page.tsx` SSR | US-3.4 (Compose Box Accessibility) |
| F4: Single-Page UI — Note List | Initial `GET /api/notes` fetch on mount; note cards rendered in newest-first order | SPEC-006 (`NoteList.tsx` fetchNotes on mount) | US-4.1 (View All Notes on Page Load) |
| F4: Single-Page UI — Note List | Note card: title (or "Untitled" fallback), body, `<time dateTime={note.created_at}>` relative timestamp; Edit/Delete buttons | SPEC-007 (`NoteCard.tsx` read view) | US-4.1 (View All Notes on Page Load) |
| F4: Single-Page UI — Note List | Empty state: "No notes yet — add your first one above." when array is empty | SPEC-006 (`NoteList.tsx` empty state branch) | US-4.2 (View Empty State When No Notes Exist) |
| F4: Single-Page UI — Note List | List refresh (re-fetch `GET /api/notes`) after create (F3 signal), edit, or delete (F5 signal) — no full page reload | SPEC-006 (`NoteList.tsx` refresh/onRefresh); SPEC-007 (`NoteCard.tsx` onRefresh prop) | US-4.3 (Note List Updates Without Page Reload) |
| F4: Single-Page UI — Note List | On `GET /api/notes` failure: inline error "Failed to load notes. Refresh to try again."; compose box unaffected | SPEC-006 (`NoteList.tsx` error state) | US-4.3 (Note List Updates Without Page Reload) |
| F5: Single-Page UI — Inline Edit & Delete | "Edit" button → inline edit mode with pre-populated title input and body textarea | SPEC-007 (`NoteCard.tsx` isEditing state; editTitle, editBody) | US-5.1 (Edit a Note Inline) |
| F5: Single-Page UI — Inline Edit & Delete | "Save" button (disabled when editBody.trim() === ''); calls `PUT /api/notes/:id`; on 200: exit edit mode, update card from response | SPEC-007 (`NoteCard.tsx` PUT save flow) | US-5.1 (Edit a Note Inline); US-5.3 (Handle Inline Edit Errors) |
| F5: Single-Page UI — Inline Edit & Delete | Only one card in edit mode at a time; second "Edit" click auto-cancels first card (unsaved changes discarded) | SPEC-007 (`NoteCard.tsx` isEditing constraint) | US-5.1 (Edit a Note Inline) |
| F5: Single-Page UI — Inline Edit & Delete | "Cancel" → revert to read view with exact original values; no API call | SPEC-007 (`NoteCard.tsx` cancel revert) | US-5.2 (Cancel an Inline Edit) |
| F5: Single-Page UI — Inline Edit & Delete | On PUT non-200 / network error: card stays in edit mode; inline error "Failed to save. Please try again."; 404 shows "Note not found. It may have been deleted." | SPEC-007 (`NoteCard.tsx` error state) | US-5.3 (Handle Inline Edit Errors) |
| F5: Single-Page UI — Inline Edit & Delete | "Delete" button → confirmation prompt "Are you sure you want to delete this note?"; on confirm: `DELETE /api/notes/:id`; on 204: remove card; on error: restore card | SPEC-007 (`NoteCard.tsx` DELETE confirm flow; optional optimistic removal) | US-5.4 (Delete a Note with Confirmation) |
| F5: Single-Page UI — Inline Edit & Delete | All edit/delete controls Tab-navigable; activatable via Enter/Space; logical focus order within card | SPEC-007 (`NoteCard.tsx` accessibility) | US-5.5 (Edit and Delete Controls are Keyboard Accessible) |

---

## 4. Requirements Detail

### 4.1 F0: Application Scaffold & Health Endpoint

**PRD Priority:** P0 — Critical  
**FRD Reference:** F00-scaffold-health  
**Phase:** 1 — Scaffold & Health  

**Functional Requirements:**
- Next.js project scaffolded with App Router and TypeScript; `tsconfig.json` with `"strict": true`
- `next.config.js` uses `.js` extension (not `.ts`) for Next.js 14 compatibility
- Dev server starts via `npm run dev`; binds to `0.0.0.0:3000`
- `GET /health` Route Handler at `app/api/health/route.ts` returns HTTP 200 `{ "status": "ok" }` with `Content-Type: application/json`
- Health endpoint has zero database dependency; responds in < 100 ms
- Wrong HTTP method on `/health` returns 405 (Next.js automatic)
- No Docker or external runtime dependency required for startup

**TechArch Implementation:**
- ADR-001: single-process monolith
- ADR-005: `next.config.js` `.js` extension
- SPEC-002: `app/api/health/route.ts`
- SPEC-009: `types/index.ts` — `HealthResponse` interface

**User Stories:** US-0.1, US-0.2

---

### 4.2 F1: Database Schema & Connectivity

**PRD Priority:** P0 — Critical  
**FRD Reference:** F01-database-schema-connectivity  
**Phase:** 2 — DB + Schema  
**Depends On:** F0  

**Functional Requirements:**
- `DATABASE_URL` environment variable read at module initialisation; absent/empty → exit code 1 with `"FATAL: DATABASE_URL environment variable is not set."`
- `PIVOTA_DB_MODE` accepted without error; direct-connect native mode in v1
- Singleton `pg.Pool` (or Prisma client) initialised from `DATABASE_URL`; shared across all Route Handlers
- Idempotent migration DDL executed at startup:
  - `CREATE EXTENSION IF NOT EXISTS pgcrypto`
  - `CREATE TABLE IF NOT EXISTS notes (id UUID NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY, title TEXT, body TEXT NOT NULL, created_at TIMESTAMPTZ NOT NULL DEFAULT now(), updated_at TIMESTAMPTZ NOT NULL DEFAULT now())`
  - `CREATE INDEX IF NOT EXISTS idx_notes_created_at ON notes (created_at DESC)`
- DB unreachable at startup → exit code 1 with `"FATAL: Cannot connect to database: <pg error>"` (password stripped)
- Migration failure → exit code 1 with `"FATAL: Migration failed: <pg error>"`
- `notes` table already exists (normal restart) → no-op, no error

**TechArch Implementation:**
- SPEC-001: `lib/db.ts` (singleton pool + startup migration)
- SPEC-008: `notes` table DDL and indexes
- ADR-002: `pg` node-postgres with parameterised SQL
- ADR-003: idempotent startup migrations (no Docker Compose)

**User Stories:** US-1.1, US-1.2, US-1.3

---

### 4.3 F2: Notes REST API

**PRD Priority:** P0 — Critical  
**FRD Reference:** F02-notes-rest-api  
**Phase:** 3 — API Routes  
**Depends On:** F1  

**Functional Requirements:**
- `GET /api/notes` → `SELECT id, title, body, created_at, updated_at FROM notes ORDER BY created_at DESC`; 200 + JSON array; `[]` on no rows; 500 on DB error
- `POST /api/notes` → validate `body` (non-null, non-empty after trim → 400 `"body is required"`); validate JSON (400 `"Invalid JSON body"`); normalise `title` (null/empty → NULL); `INSERT INTO notes (title, body) VALUES ($1, $2) RETURNING *`; 201 + created note
- `GET /api/notes/:id` → `SELECT … FROM notes WHERE id = $1`; 200 + note; 404 `"Note not found"` if no row; 500 on DB error
- `PUT /api/notes/:id` → same body validation as POST; `UPDATE notes SET title = $1, body = $2, updated_at = now() WHERE id = $3 RETURNING *`; 200 + updated note; 404 if 0 rows; 400 on validation fail
- `DELETE /api/notes/:id` → `DELETE FROM notes WHERE id = $1 RETURNING id`; 204 no body; 404 if 0 rows; 500 on DB error
- All errors return `{ "error": "<message>" }` JSON; wrong method returns 405 automatically
- All SQL uses parameterised queries (`$1`, `$2`, …) — no string interpolation

**TechArch Implementation:**
- SPEC-003: `app/api/notes/route.ts` (GET + POST)
- SPEC-004: `app/api/notes/[id]/route.ts` (GET + PUT + DELETE)
- SPEC-008: `notes` table (reads/writes all five columns)
- ADR-002: `pg` parameterised SQL; SQL injection prevention

**User Stories:** US-2.1, US-2.2, US-2.3, US-2.4, US-2.5

---

### 4.4 F3: Single-Page UI — Compose Box

**PRD Priority:** P0 — Critical  
**FRD Reference:** F03-compose-box-ui  
**Phase:** 4 — UI  
**Depends On:** F2  

**Functional Requirements:**
- Page `<h1>` (or equivalent) displays "QuickNotes"
- Compose box rendered in initial SSR HTML via `app/page.tsx`
- Title: `<input type="text">` with placeholder `"Title (optional)"`, associated `<label>`, not required
- Body: `<textarea>` with placeholder `"Write a note…"`, associated `<label>`, functionally required
- "Add Note" `<button>` has native HTML `disabled` attribute when `body.trim() === ""` or `isSubmitting === true`
- On submit: `POST /api/notes` with `{ title: title.trim() || null, body: body.trim() }`
- On 201: clear title + body to `""`; call `onNoteCreated()` to trigger list refresh
- On non-201 / network error: display inline error "Failed to save note. Please try again."; form NOT cleared
- All controls Tab-navigable; labels use `htmlFor`/`id` pairing

**TechArch Implementation:**
- SPEC-005: `app/components/ComposeBox.tsx` (local state: title, body, isSubmitting, error)
- `app/page.tsx`: SSR shell rendering ComposeBox + NoteList

**User Stories:** US-3.1, US-3.2, US-3.3, US-3.4

---

### 4.5 F4: Single-Page UI — Note List

**PRD Priority:** P0 — Critical  
**FRD Reference:** F04-note-list-ui  
**Phase:** 4 — UI  
**Depends On:** F2, F3  

**Functional Requirements:**
- `GET /api/notes` called on component mount; notes rendered in order returned (newest first)
- Note card displays: title (`note.title` or `"Untitled"` if null/empty/whitespace), body, relative timestamp via `<time dateTime={note.created_at}>`
- Empty state: renders `<p>No notes yet — add your first one above.</p>` when array is empty
- List refresh (re-fetch `GET /api/notes`) triggered after: F3 POST success, F5 PUT success, F5 DELETE success
- No full page reload required for any list update
- On `GET /api/notes` failure: inline error "Failed to load notes. Refresh to try again."; compose box unaffected
- Compose box must remain fully visible and usable during list fetch (no loading overlay)

**TechArch Implementation:**
- SPEC-006: `app/components/NoteList.tsx` (notes state, isLoading, error; fetchNotes(); refresh())
- SPEC-007: `app/components/NoteCard.tsx` (read view portion)

**User Stories:** US-4.1, US-4.2, US-4.3

---

### 4.6 F5: Single-Page UI — Inline Edit & Delete

**PRD Priority:** P1 — High  
**FRD Reference:** F05-inline-edit-delete-ui  
**Phase:** 4 — UI + Polish  
**Depends On:** F2, F4  

**Functional Requirements:**
- "Edit" button on each note card → transition to edit mode: title `<input>` pre-filled with `note.title || ""`; body `<textarea>` pre-filled with `note.body`; "Save" and "Cancel" buttons
- "Save" `<button>` has native `disabled` attribute when `editBody.trim() === ""`
- On Save: `PUT /api/notes/:id` with `{ title: editTitle.trim() || null, body: editBody.trim() }`; on 200: exit edit mode, update card from response, call `onRefresh()`
- On PUT error: card remains in edit mode; inline error "Failed to save. Please try again."; on 404: "Note not found. It may have been deleted."
- "Cancel" → revert card to read view with exact original values; no API call
- Only one card in edit mode at a time; activating "Edit" on a second card auto-cancels the first (no confirmation)
- "Delete" button → `window.confirm("Are you sure you want to delete this note?")`; on cancel: no action
- On confirm delete: `DELETE /api/notes/:id`; on 204: remove card from list; if last note, empty state appears
- On DELETE 404: silently remove card (note already gone)
- On DELETE 500 / network error: restore card (if optimistically hidden); inline error "Failed to delete note. Please try again."
- All controls (Edit, Delete, Save, Cancel) Tab-navigable and activatable via Enter/Space

**TechArch Implementation:**
- SPEC-007: `app/components/NoteCard.tsx` (isEditing, editTitle, editBody, isSaving, error state; PUT save flow; DELETE confirm flow; onRefresh prop)

**User Stories:** US-5.1, US-5.2, US-5.3, US-5.4, US-5.5

---

## 5. Test Case Coverage Matrix

Test cases are derived from the acceptance criteria of each User Story. Each test case maps to the user story (US), the PRD feature (F), the TechArch spec (SPEC/ADR), and the acceptance criterion validated.

### 5.1 F0: Application Scaffold & Health Endpoint

| Test ID | Test Case Description | US Ref | PRD Feature | TechArch Spec | Type | Priority |
|---------|-----------------------|--------|-------------|---------------|------|----------|
| TEST-001 | `npm run dev` starts the server without errors | US-0.1 | F0 | ADR-001 | Integration | P0 |
| TEST-002 | Server binds to `0.0.0.0:3000` (not `127.0.0.1`) | US-0.1 | F0 | ADR-001 | Integration | P0 |
| TEST-003 | App is reachable at `http://localhost:3000` from browser | US-0.1 | F0 | ADR-001 | E2E | P0 |
| TEST-004 | `next.config.js` uses `.js` extension (not `.ts`) | US-0.1 | F0 | ADR-005 | Static | P0 |
| TEST-005 | `tsconfig.json` has `"strict": true` enabled | US-0.1 | F0 | SPEC-009 | Static | P0 |
| TEST-006 | App starts without Docker or external runtime dependency | US-0.1 | F0 | ADR-001 | Integration | P0 |
| TEST-007 | `GET /health` returns HTTP 200 | US-0.2 | F0 | SPEC-002 | API | P0 |
| TEST-008 | `GET /health` response body is exactly `{ "status": "ok" }` | US-0.2 | F0 | SPEC-002 | API | P0 |
| TEST-009 | `GET /health` `Content-Type` is `application/json` | US-0.2 | F0 | SPEC-002 | API | P0 |
| TEST-010 | `GET /health` responds in under 100 ms | US-0.2 | F0 | SPEC-002 | Performance | P0 |
| TEST-011 | `GET /health` returns 200 even when database is unreachable | US-0.2 | F0 | SPEC-002 | Integration | P0 |
| TEST-012 | `POST /health` returns HTTP 405 | US-0.2 | F0 | SPEC-002 | API | P0 |

### 5.2 F1: Database Schema & Connectivity

| Test ID | Test Case Description | US Ref | PRD Feature | TechArch Spec | Type | Priority |
|---------|-----------------------|--------|-------------|---------------|------|----------|
| TEST-013 | App reads `DATABASE_URL` at module init (not lazily) | US-1.1 | F1 | SPEC-001 | Integration | P0 |
| TEST-014 | Singleton connection pool is established using `DATABASE_URL` | US-1.1 | F1 | SPEC-001 | Integration | P0 |
| TEST-015 | Pool is reused across multiple Route Handler invocations | US-1.1 | F1 | SPEC-001 | Integration | P0 |
| TEST-016 | `PIVOTA_DB_MODE` env var is accepted without error | US-1.1 | F1 | SPEC-001 | Integration | P0 |
| TEST-017 | App does not require Docker to connect to database | US-1.1 | F1 | ADR-003 | Integration | P0 |
| TEST-018 | `notes` table is created if it does not exist | US-1.2 | F1 | SPEC-008 | Integration | P0 |
| TEST-019 | `notes` table has correct column schema (id, title, body, created_at, updated_at) | US-1.2 | F1 | SPEC-008 | Integration | P0 |
| TEST-020 | Migration DDL is idempotent — restarting multiple times causes no schema error | US-1.2 | F1 | SPEC-008; ADR-003 | Integration | P0 |
| TEST-021 | `pgcrypto` extension enabled via `CREATE EXTENSION IF NOT EXISTS pgcrypto` | US-1.2 | F1 | SPEC-008 | Integration | P0 |
| TEST-022 | Index `idx_notes_created_at` created via `CREATE INDEX IF NOT EXISTS` | US-1.2 | F1 | SPEC-008 | Integration | P0 |
| TEST-023 | Migration runs natively via `DATABASE_URL` without Docker Compose | US-1.2 | F1 | ADR-003 | Integration | P0 |
| TEST-024 | Absent `DATABASE_URL` → process exits with code 1 and logs `"FATAL: DATABASE_URL environment variable is not set."` | US-1.3 | F1 | SPEC-001 | Integration | P0 |
| TEST-025 | Unreachable DB at startup → exit code 1 and logs `"FATAL: Cannot connect to database: <pg error>"` | US-1.3 | F1 | SPEC-001 | Integration | P0 |
| TEST-026 | Migration DDL failure (permissions) → exit code 1 and logs `"FATAL: Migration failed: <pg error>"` | US-1.3 | F1 | SPEC-001 | Integration | P0 |
| TEST-027 | Full `DATABASE_URL` (including password) is never logged in any error output | US-1.3 | F1 | SPEC-001 | Security | P0 |
| TEST-028 | Malformed `DATABASE_URL` → exit code 1 with pg parse error included | US-1.3 | F1 | SPEC-001 | Integration | P0 |

### 5.3 F2: Notes REST API

| Test ID | Test Case Description | US Ref | PRD Feature | TechArch Spec | Type | Priority |
|---------|-----------------------|--------|-------------|---------------|------|----------|
| TEST-029 | `GET /api/notes` returns HTTP 200 with a JSON array | US-2.1 | F2 | SPEC-003 | API | P0 |
| TEST-030 | Notes returned by `GET /api/notes` are ordered by `created_at` descending | US-2.1 | F2 | SPEC-003 | API | P0 |
| TEST-031 | Each note object in `GET /api/notes` includes id, title, body, created_at, updated_at | US-2.1 | F2 | SPEC-003 | API | P0 |
| TEST-032 | `GET /api/notes` returns empty array `[]` when no notes exist | US-2.1 | F2 | SPEC-003 | API | P0 |
| TEST-033 | `GET /api/notes` returns 500 `{ "error": "Internal server error" }` on DB failure | US-2.1 | F2 | SPEC-003 | API | P0 |
| TEST-034 | `POST /api/notes` with `{ title, body }` returns HTTP 201 with created note | US-2.2 | F2 | SPEC-003 | API | P0 |
| TEST-035 | Created note includes auto-generated `id`, `created_at`, and `updated_at` | US-2.2 | F2 | SPEC-003; SPEC-008 | API | P0 |
| TEST-036 | `POST /api/notes`: `title` stored as NULL if absent, null, or empty string after trim | US-2.2 | F2 | SPEC-003 | API | P0 |
| TEST-037 | `POST /api/notes` with missing/empty `body` returns HTTP 400 `{ "error": "body is required" }` | US-2.2 | F2 | SPEC-003 | API | P0 |
| TEST-038 | `POST /api/notes` with malformed JSON returns HTTP 400 `{ "error": "Invalid JSON body" }` | US-2.2 | F2 | SPEC-003 | API | P0 |
| TEST-039 | `POST /api/notes` returns 500 `{ "error": "Internal server error" }` on DB failure | US-2.2 | F2 | SPEC-003 | API | P0 |
| TEST-040 | `GET /api/notes/:id` with valid existing ID returns HTTP 200 with note object | US-2.3 | F2 | SPEC-004 | API | P0 |
| TEST-041 | `GET /api/notes/:id` with non-existent ID returns HTTP 404 `{ "error": "Note not found" }` | US-2.3 | F2 | SPEC-004 | API | P0 |
| TEST-042 | `GET /api/notes/:id` with malformed UUID returns 404 (not 400) | US-2.3 | F2 | SPEC-004 | API | P0 |
| TEST-043 | `GET /api/notes/:id` returns 500 `{ "error": "Internal server error" }` on DB failure | US-2.3 | F2 | SPEC-004 | API | P0 |
| TEST-044 | `PUT /api/notes/:id` with `{ title, body }` returns HTTP 200 with updated note | US-2.4 | F2 | SPEC-004 | API | P0 |
| TEST-045 | `PUT /api/notes/:id` bumps `updated_at` to `now()` on every successful update | US-2.4 | F2 | SPEC-004; SPEC-008 | API | P0 |
| TEST-046 | `PUT /api/notes/:id`: `title` normalisation matches POST (null/empty → NULL) | US-2.4 | F2 | SPEC-004 | API | P0 |
| TEST-047 | `PUT /api/notes/:id` with missing/empty `body` returns HTTP 400 `{ "error": "body is required" }` | US-2.4 | F2 | SPEC-004 | API | P0 |
| TEST-048 | `PUT /api/notes/:id` with non-existent ID returns HTTP 404 `{ "error": "Note not found" }` | US-2.4 | F2 | SPEC-004 | API | P0 |
| TEST-049 | `PUT /api/notes/:id` with malformed JSON returns HTTP 400 `{ "error": "Invalid JSON body" }` | US-2.4 | F2 | SPEC-004 | API | P0 |
| TEST-050 | `DELETE /api/notes/:id` with valid existing ID returns HTTP 204 with no response body | US-2.5 | F2 | SPEC-004 | API | P0 |
| TEST-051 | Deleted note no longer appears in subsequent `GET /api/notes` response | US-2.5 | F2 | SPEC-004 | API | P0 |
| TEST-052 | `DELETE /api/notes/:id` with non-existent ID returns HTTP 404 `{ "error": "Note not found" }` | US-2.5 | F2 | SPEC-004 | API | P0 |
| TEST-053 | `DELETE /api/notes/:id` returns 500 `{ "error": "Internal server error" }` on DB failure | US-2.5 | F2 | SPEC-004 | API | P0 |

### 5.4 F3: Single-Page UI — Compose Box

| Test ID | Test Case Description | US Ref | PRD Feature | TechArch Spec | Type | Priority |
|---------|-----------------------|--------|-------------|---------------|------|----------|
| TEST-054 | Page header displays "QuickNotes" as `<h1>` or equivalent | US-3.1 | F3 | SPEC-005 | E2E | P0 |
| TEST-055 | Compose box is visible at the top of the page on initial load without navigation | US-3.1 | F3 | SPEC-005 | E2E | P0 |
| TEST-056 | Title input is `<input type="text">` with placeholder "Title (optional)" and is not required | US-3.1 | F3 | SPEC-005 | E2E | P0 |
| TEST-057 | Body textarea is `<textarea>` with placeholder "Write a note…" | US-3.1 | F3 | SPEC-005 | E2E | P0 |
| TEST-058 | Clicking "Add Note" calls `POST /api/notes` with `{ title: trimmedTitle \|\| null, body: trimmedBody }` | US-3.1 | F3 | SPEC-005 | E2E | P0 |
| TEST-059 | On 201 success, form is cleared and note list is refreshed | US-3.1 | F3 | SPEC-005 | E2E | P0 |
| TEST-060 | Note creation round-trip (POST + UI update) completes in under 500 ms on local network | US-3.1 | F3 | SPEC-005 | Performance | P0 |
| TEST-061 | "Add Note" button has native `disabled` attribute when `body.trim() === ""` | US-3.2 | F3 | SPEC-005 | E2E | P0 |
| TEST-062 | Button becomes enabled when body contains at least one non-whitespace character | US-3.2 | F3 | SPEC-005 | E2E | P0 |
| TEST-063 | Whitespace-only body (spaces, tabs, newlines) keeps button disabled | US-3.2 | F3 | SPEC-005 | E2E | P0 |
| TEST-064 | Client-side guard prevents submission even if `disabled` attribute is bypassed | US-3.2 | F3 | SPEC-005 | Unit | P0 |
| TEST-065 | Server returns HTTP 400 if empty body reaches the API | US-3.2 | F3 | SPEC-003 | API | P0 |
| TEST-066 | Non-201 response shows inline error "Failed to save note. Please try again." | US-3.3 | F3 | SPEC-005 | E2E | P0 |
| TEST-067 | Form is NOT cleared on error — title and body values are preserved | US-3.3 | F3 | SPEC-005 | E2E | P0 |
| TEST-068 | Network error (fetch throws) shows same inline error message | US-3.3 | F3 | SPEC-005 | E2E | P0 |
| TEST-069 | Button returns to normal enabled/disabled state after error (not stuck in loading) | US-3.3 | F3 | SPEC-005 | E2E | P0 |
| TEST-070 | Title input has associated `<label>` via `htmlFor`/`id` pairing | US-3.4 | F3 | SPEC-005 | Accessibility | P0 |
| TEST-071 | Body textarea has associated `<label>` via `htmlFor`/`id` pairing | US-3.4 | F3 | SPEC-005 | Accessibility | P0 |
| TEST-072 | All compose box controls reachable via Tab key | US-3.4 | F3 | SPEC-005 | Accessibility | P0 |
| TEST-073 | "Add Note" button activatable via Enter or Space when focused | US-3.4 | F3 | SPEC-005 | Accessibility | P0 |
| TEST-074 | Compose box is present in initial server-side HTML (SSR) | US-3.4 | F3 | SPEC-005 | E2E | P0 |

### 5.5 F4: Single-Page UI — Note List

| Test ID | Test Case Description | US Ref | PRD Feature | TechArch Spec | Type | Priority |
|---------|-----------------------|--------|-------------|---------------|------|----------|
| TEST-075 | Note list populated by `GET /api/notes` on page load | US-4.1 | F4 | SPEC-006 | E2E | P0 |
| TEST-076 | Each note card displays title (or "Untitled"), body, and relative timestamp | US-4.1 | F4 | SPEC-007 | E2E | P0 |
| TEST-077 | Notes displayed in newest-first order (as returned by API) | US-4.1 | F4 | SPEC-006 | E2E | P0 |
| TEST-078 | "Untitled" fallback for null, empty string, or whitespace-only titles | US-4.1 | F4 | SPEC-007 | E2E | P0 |
| TEST-079 | Relative timestamp rendered with `<time dateTime={note.created_at}>` element | US-4.1 | F4 | SPEC-007 | E2E | P0 |
| TEST-080 | Page initial load with notes list completes in under 2 seconds | US-4.1 | F4 | SPEC-006 | Performance | P0 |
| TEST-081 | Empty array from `GET /api/notes` shows "No notes yet — add your first one above." | US-4.2 | F4 | SPEC-006 | E2E | P0 |
| TEST-082 | Empty state message matches exactly (no paraphrasing) | US-4.2 | F4 | SPEC-006 | E2E | P0 |
| TEST-083 | Compose box remains fully usable while empty state is shown | US-4.2 | F4 | SPEC-006 | E2E | P0 |
| TEST-084 | First note created: empty state disappears; note appears in list without page reload | US-4.2 | F4 | SPEC-006; SPEC-005 | E2E | P0 |
| TEST-085 | Last note deleted: empty state reappears without page reload | US-4.2 | F4 | SPEC-006; SPEC-007 | E2E | P0 |
| TEST-086 | Creating a note triggers list re-fetch without full page reload | US-4.3 | F4 | SPEC-006; SPEC-005 | E2E | P0 |
| TEST-087 | Newly created note appears at top of list after submission | US-4.3 | F4 | SPEC-006 | E2E | P0 |
| TEST-088 | Round-trip from note submission to card visible in list < 500 ms | US-4.3 | F4 | SPEC-006 | Performance | P0 |
| TEST-089 | Editing a note updates card in place without full page reload | US-4.3 | F4 | SPEC-007 | E2E | P0 |
| TEST-090 | Deleting a note removes card from list without full page reload | US-4.3 | F4 | SPEC-007 | E2E | P0 |
| TEST-091 | `GET /api/notes` failure on refresh: previous list state retained; inline error shown | US-4.3 | F4 | SPEC-006 | E2E | P0 |

### 5.6 F5: Single-Page UI — Inline Edit & Delete

| Test ID | Test Case Description | US Ref | PRD Feature | TechArch Spec | Type | Priority |
|---------|-----------------------|--------|-------------|---------------|------|----------|
| TEST-092 | Clicking "Edit" transitions card to edit mode with pre-filled title input and body textarea | US-5.1 | F5 | SPEC-007 | E2E | P1 |
| TEST-093 | "Save" and "Cancel" buttons replace "Edit" and "Delete" in edit mode | US-5.1 | F5 | SPEC-007 | E2E | P1 |
| TEST-094 | "Save" button has native `disabled` attribute when `body.trim() === ""` in edit mode | US-5.1 | F5 | SPEC-007 | E2E | P1 |
| TEST-095 | Clicking "Save" calls `PUT /api/notes/:id` with trimmed title and body | US-5.1 | F5 | SPEC-007 | E2E | P1 |
| TEST-096 | On 200, card exits edit mode and displays updated title, body, timestamp from API response | US-5.1 | F5 | SPEC-007 | E2E | P1 |
| TEST-097 | Only one card in edit mode at a time; second "Edit" auto-cancels first (no confirmation) | US-5.1 | F5 | SPEC-007 | E2E | P1 |
| TEST-098 | Clicking "Cancel" returns card to read view without API call | US-5.2 | F5 | SPEC-007 | E2E | P1 |
| TEST-099 | Card displays exact original title and body after Cancel (not trimmed/modified) | US-5.2 | F5 | SPEC-007 | E2E | P1 |
| TEST-100 | No inline error or loading state triggered by Cancel | US-5.2 | F5 | SPEC-007 | E2E | P1 |
| TEST-101 | "Edit" and "Delete" buttons reappear after Cancel | US-5.2 | F5 | SPEC-007 | E2E | P1 |
| TEST-102 | PUT non-200 response: card stays in edit mode with inline error "Failed to save. Please try again." | US-5.3 | F5 | SPEC-007 | E2E | P1 |
| TEST-103 | PUT 404 response: inline error "Note not found. It may have been deleted." | US-5.3 | F5 | SPEC-007 | E2E | P1 |
| TEST-104 | Network error on PUT: same inline error as non-200 | US-5.3 | F5 | SPEC-007 | E2E | P1 |
| TEST-105 | "Save" and "Cancel" buttons return to normal state after error | US-5.3 | F5 | SPEC-007 | E2E | P1 |
| TEST-106 | Clicking "Delete" shows confirmation prompt "Are you sure you want to delete this note?" | US-5.4 | F5 | SPEC-007 | E2E | P1 |
| TEST-107 | Cancelling confirmation: no API call, card unchanged | US-5.4 | F5 | SPEC-007 | E2E | P1 |
| TEST-108 | Confirming delete calls `DELETE /api/notes/:id` | US-5.4 | F5 | SPEC-007 | E2E | P1 |
| TEST-109 | On 204, note card is removed from the list | US-5.4 | F5 | SPEC-007 | E2E | P1 |
| TEST-110 | Deleting last note causes empty state message to appear | US-5.4 | F5 | SPEC-007 | E2E | P1 |
| TEST-111 | DELETE 500 or network error: card restored (if optimistically removed); inline error shown | US-5.4 | F5 | SPEC-007 | E2E | P1 |
| TEST-112 | DELETE 404: card silently removed from list (no error shown to user) | US-5.4 | F5 | SPEC-007 | E2E | P1 |
| TEST-113 | "Edit" and "Delete" buttons on each card reachable via Tab key | US-5.5 | F5 | SPEC-007 | Accessibility | P1 |
| TEST-114 | "Edit" and "Delete" buttons activatable via Enter or Space | US-5.5 | F5 | SPEC-007 | Accessibility | P1 |
| TEST-115 | In edit mode, "Save" and "Cancel" buttons are Tab-navigable and keyboard-activatable | US-5.5 | F5 | SPEC-007 | Accessibility | P1 |
| TEST-116 | Focus order within note card is logical (title → body → Save/Cancel or Edit/Delete) | US-5.5 | F5 | SPEC-007 | Accessibility | P1 |

### 5.7 Coverage Summary

| Feature | User Stories | Test Cases | Coverage |
|---------|-------------|------------|----------|
| F0: Scaffold & Health | US-0.1, US-0.2 (2 stories) | TEST-001 – TEST-012 (12 tests) | 100% |
| F1: Database Schema & Connectivity | US-1.1, US-1.2, US-1.3 (3 stories) | TEST-013 – TEST-028 (16 tests) | 100% |
| F2: Notes REST API | US-2.1, US-2.2, US-2.3, US-2.4, US-2.5 (5 stories) | TEST-029 – TEST-053 (25 tests) | 100% |
| F3: Compose Box UI | US-3.1, US-3.2, US-3.3, US-3.4 (4 stories) | TEST-054 – TEST-074 (21 tests) | 100% |
| F4: Note List UI | US-4.1, US-4.2, US-4.3 (3 stories) | TEST-075 – TEST-091 (17 tests) | 100% |
| F5: Inline Edit & Delete UI | US-5.1, US-5.2, US-5.3, US-5.4, US-5.5 (5 stories) | TEST-092 – TEST-116 (25 tests) | 100% |
| **Total** | **22 stories** | **116 tests** | **100%** |

### 5.8 Test Case Types Summary

| Test Type | Count | Features Covered |
|-----------|-------|-----------------|
| API | 29 | F0, F2 |
| E2E | 57 | F3, F4, F5 |
| Integration | 21 | F1 |
| Performance | 4 | F0, F3, F4 |
| Accessibility | 8 | F3, F5 |
| Security | 1 | F1 |
| Static | 2 | F0 |
| Unit | 1 | F3 |
| **Total** | **116** | **F0–F5** |

---

## 6. Change Management

### 6.1 Change Log

| Change ID | Date | Version | Changed By | Section | Description | Impact |
|-----------|------|---------|------------|---------|-------------|--------|
| CHG-001 | 2026-07-04 | 1.0 | System | All | Initial RTM created from PRD v1.0, FRD v1.0, TechArch v1.0, UserStories v1.0 | Baseline document — no prior version |

### 6.2 Change Control Process

Any modification to a source specification document (PRD, FRD, TechArch, UserStories) that adds, removes, or changes a requirement, spec, or acceptance criterion must trigger an RTM update in the same change set. The RTM version must be bumped to match the highest version of any updated source document. All traceability links affected by the change must be reviewed and updated before the change is marked complete.

---

## 7. Validation Checklist

### 7.1 Forward Traceability (PRD → Test)

| Check | Status |
|-------|--------|
| All PRD features (F0–F5) have at least one FRD sub-feature | ✅ Complete |
| All PRD features (F0–F5) have at least one TechArch spec (SPEC or ADR) | ✅ Complete |
| All PRD features (F0–F5) have at least one User Story | ✅ Complete |
| All PRD features (F0–F5) have at least one test case | ✅ Complete |
| All User Story acceptance criteria have a corresponding test case | ✅ Complete |

### 7.2 Backward Traceability (Test → PRD)

| Check | Status |
|-------|--------|
| All test cases map to a User Story | ✅ Complete |
| All User Stories map to a PRD Feature | ✅ Complete |
| All TechArch specs (SPEC-001–SPEC-009, ADR-001–ADR-005) map to at least one FRD sub-feature | ✅ Complete |
| All FRD sub-features map to a PRD Feature | ✅ Complete |

### 7.3 ID Consistency

| Check | Status |
|-------|--------|
| PRD feature IDs follow F0–F5 convention | ✅ Verified |
| FRD sub-features reference parent F0–F5 feature | ✅ Verified |
| TechArch SPEC-001–SPEC-009 IDs are consistent with component names | ✅ Verified |
| ADR-001–ADR-005 IDs match TechArch Appendix A | ✅ Verified |
| User Story IDs follow US-[epic].[sequence] convention (US-0.1–US-5.5) | ✅ Verified |
| Test Case IDs follow TEST-NNN sequential convention (TEST-001–TEST-116) | ✅ Verified |

---

## 8. Approval

| Role | Name | Signature | Date | Status |
|------|------|-----------|------|--------|
| Product Owner | — | — | — | Pending |
| Lead Engineer | — | — | — | Pending |
| QA Lead | — | — | — | Pending |
| Project Sponsor | — | — | — | Pending |

---

*Document generated: 2026-07-04*  
*Based on: PRD-quicknotes2.md v1.0, FRD-quicknotes2.md v1.0, TechArch-quicknotes2.md v1.0, UserStories-quicknotes2.md v1.0, .planning/PROJECT.md*  
*Next review: on any change to a source specification document*
