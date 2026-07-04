# Roadmap: QuickNotes

## Overview

QuickNotes is built in five natural delivery phases, each corresponding directly to a feature group from the requirements. The phases follow a strict dependency chain: scaffold enables database, database enables API, API enables UI, and the final UI layer (inline edit/delete) layers on top of the core list view. Every phase delivers a verifiable, runnable capability — nothing hangs incomplete waiting for the next phase.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Scaffold & Health** - Next.js App Router project starts cleanly and responds to health checks
- [ ] **Phase 2: Database Schema & Connectivity** - App connects to PostgreSQL and migrates the `notes` table at startup
- [ ] **Phase 3: Notes REST API** - Full CRUD API for notes over HTTP/JSON
- [ ] **Phase 4: Core UI** - Compose box and note list render and function end-to-end
- [ ] **Phase 5: Inline Edit & Delete** - Users can edit and delete notes directly on the page

## Phase Details

### Phase 1: Scaffold & Health
**Goal**: The Next.js application starts and is verifiably alive
**Depends on**: Nothing (first phase)
**Requirements**: F0-01, F0-02, F0-03
**Success Criteria** (what must be TRUE):
  1. Running `npm run dev` starts the server on port 3000 bound to `0.0.0.0` with no errors
  2. `GET /health` returns HTTP 200 with JSON body `{ "status": "ok" }` within 100ms
  3. The project uses Next.js App Router with TypeScript (`tsconfig.json` strict mode, `next.config.js` with `.js` extension)
**Plans**: 1 plan

Plans:
- [ ] 01-01-PLAN.md — Scaffold Next.js App Router + TypeScript + GET /health route

### Phase 2: Database Schema & Connectivity
**Goal**: The app connects to PostgreSQL and ensures the schema is ready before serving requests
**Depends on**: Phase 1
**Requirements**: F1-01, F1-02, F1-03
**Success Criteria** (what must be TRUE):
  1. App starts successfully when `DATABASE_URL` is set and PostgreSQL is reachable
  2. App exits immediately with a clear `FATAL:` error message when `DATABASE_URL` is missing or DB is unreachable
  3. The `notes` table exists with correct columns (`id`, `title`, `body`, `created_at`, `updated_at`) after startup — and startup is idempotent (safe to run multiple times)
**Plans**: TBD

### Phase 3: Notes REST API
**Goal**: All five CRUD endpoints for notes work correctly and return proper HTTP status codes
**Depends on**: Phase 2
**Requirements**: F2-01, F2-02, F2-03, F2-04, F2-05
**Success Criteria** (what must be TRUE):
  1. `GET /api/notes` returns a JSON array of notes ordered newest first (empty array `[]` when none exist)
  2. `POST /api/notes` creates a note and returns 201 with the created note; returns 400 when `body` is empty
  3. `GET /api/notes/:id` returns the note or 404 if not found
  4. `PUT /api/notes/:id` updates the note and bumps `updated_at`; returns 404 if not found
  5. `DELETE /api/notes/:id` deletes the note and returns 204; returns 404 if not found
**Plans**: TBD

### Phase 4: Core UI
**Goal**: Users can compose and view notes in the browser without page reloads
**Depends on**: Phase 3
**Requirements**: F3-01, F3-02, F3-03, F4-01, F4-02, F4-03
**Success Criteria** (what must be TRUE):
  1. Page header shows "QuickNotes"; compose box with optional title input and required body textarea is always visible
  2. "Add Note" button is disabled when the body textarea is empty; submitting a valid note clears the form and shows the new note at the top of the list without a page reload
  3. Note list displays each note with title (or "Untitled"), body, and relative timestamp, ordered newest first
  4. "No notes yet — add your first one above." appears when no notes exist
  5. Note list loads within 500ms of page render
**Plans**: TBD

### Phase 5: Inline Edit & Delete
**Goal**: Users can modify and remove existing notes directly on the page
**Depends on**: Phase 4
**Requirements**: F5-01, F5-02, F5-03, F5-04
**Success Criteria** (what must be TRUE):
  1. Clicking Edit on a note card enters inline edit mode with title and body pre-populated; saving updates the card in place
  2. Clicking Cancel in edit mode reverts the card to its original values without saving
  3. Clicking Delete shows a confirmation prompt; confirming removes the note from the list (showing empty state if it was the last)
  4. Only one note can be in edit mode at a time — opening a second auto-cancels the first
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Scaffold & Health | 0/TBD | Not started | - |
| 2. Database Schema & Connectivity | 0/TBD | Not started | - |
| 3. Notes REST API | 0/TBD | Not started | - |
| 4. Core UI | 0/TBD | Not started | - |
| 5. Inline Edit & Delete | 0/TBD | Not started | - |
