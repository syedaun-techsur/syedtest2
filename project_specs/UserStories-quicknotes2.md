# User Stories
# QuickNotes

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Version:** 1.0  
**Date:** 2026-07-04  
**Status:** Draft  
**Based On:** PRD-quicknotes2.md v1.0, FRD-quicknotes2.md v1.0, PERSONAS-quicknotes2.md v1.0  

---

## Personas

| ID | Name | Role |
|---|---|---|
| PER-01 | Alex Rivera | Knowledge Worker / Daily Note-Taker |
| PER-02 | Jordan Kim | Developer / Technical Evaluator |

---

## Priority Definitions

| Priority | Label | Meaning |
|---|---|---|
| P0 | Critical | MVP prerequisite; product cannot ship without this |
| P1 | High | Required for complete feature set; ships in MVP but layered after P0 |
| P2 | Medium | Significant value; acceptable to defer to v1.1 |
| P3 | Low | Nice to have; explicit backlog |

---

## Epic 0: Application Scaffold & Health Endpoint (F0)

_Establishes the foundational Next.js project structure. Enables Jordan to start the app with a single command and verify liveness via a health endpoint before routing any traffic._

---

### US-0.1: Start the Application
**As a** developer (Jordan Kim), **I want to** start the application with `npm run dev` and have it bind to `0.0.0.0:3000`, **so that** it is immediately reachable in sandbox and preview environments without additional network configuration.

**Acceptance Criteria:**
- [ ] Running `npm run dev` starts the server without errors
- [ ] Server binds to `0.0.0.0:3000` (not just `127.0.0.1`)
- [ ] App is reachable at `http://localhost:3000` from a browser
- [ ] `next.config.js` uses the `.js` extension (not `.ts`) for Next.js 14 compatibility
- [ ] `tsconfig.json` has `"strict": true` enabled
- [ ] No Docker or external runtime dependency is required to start the app

**Priority:** P0 | **Feature Ref:** F0

---

### US-0.2: Verify Application Health
**As a** developer (Jordan Kim), **I want to** call `GET /health` and receive a machine-readable `{ "status": "ok" }` response, **so that** I can confirm the application is live before routing traffic to it.

**Acceptance Criteria:**
- [ ] `GET /health` returns HTTP 200
- [ ] Response body is exactly `{ "status": "ok" }` with `Content-Type: application/json`
- [ ] Endpoint responds in under 100 ms
- [ ] Endpoint has no database dependency and returns 200 even if the database is unreachable
- [ ] Wrong HTTP method (e.g. `POST /health`) returns HTTP 405

**Priority:** P0 | **Feature Ref:** F0

---

## Epic 1: Database Schema & Connectivity (F1)

_Connects the application to PostgreSQL via `DATABASE_URL`, runs idempotent migrations at startup, and fails fast with a clear error if the environment is misconfigured._

---

### US-1.1: Connect to the Database at Startup
**As a** developer (Jordan Kim), **I want to** set `DATABASE_URL` in the environment and have the app connect to PostgreSQL automatically on startup, **so that** no manual database wiring step is needed after setting the environment variable.

**Acceptance Criteria:**
- [ ] App reads `DATABASE_URL` at module initialisation time (not lazily on first request)
- [ ] A singleton connection pool is established using `DATABASE_URL`
- [ ] The pool is reused across all API Route Handler invocations (not a new connection per request)
- [ ] `PIVOTA_DB_MODE` environment variable is accepted without error (no change to connection logic in v1)
- [ ] App does not require Docker or Docker Compose to connect to the database

**Priority:** P0 | **Feature Ref:** F1

---

### US-1.2: Migrate the Database Schema on Startup
**As a** developer (Jordan Kim), **I want to** have the `notes` table created automatically when the app starts, **so that** I do not need to run migrations manually and can restart the app safely without schema conflicts.

**Acceptance Criteria:**
- [ ] `notes` table is created if it does not exist (`CREATE TABLE IF NOT EXISTS`)
- [ ] Table has columns: `id` (UUID PK, auto-generated), `title` (TEXT, nullable), `body` (TEXT, not null), `created_at` (TIMESTAMPTZ, default `now()`), `updated_at` (TIMESTAMPTZ, default `now()`)
- [ ] Migration DDL is idempotent — restarting the app multiple times produces no schema errors
- [ ] `pgcrypto` extension is enabled via `CREATE EXTENSION IF NOT EXISTS pgcrypto`
- [ ] An index on `created_at DESC` is created (`CREATE INDEX IF NOT EXISTS`)
- [ ] Migration runs natively via `DATABASE_URL` — never via Docker Compose

**Priority:** P0 | **Feature Ref:** F1

---

### US-1.3: Fail Fast on Missing or Invalid Database Configuration
**As a** developer (Jordan Kim), **I want to** see an immediate, descriptive error when `DATABASE_URL` is missing or the database is unreachable, **so that** I can diagnose and fix configuration problems quickly without silent hangs or cryptic stack traces.

**Acceptance Criteria:**
- [ ] If `DATABASE_URL` is absent or empty, the process exits with code 1 and logs: `"FATAL: DATABASE_URL environment variable is not set."`
- [ ] If the database is unreachable at startup, the process exits with code 1 and logs: `"FATAL: Cannot connect to database: <pg error>"`
- [ ] If the migration DDL fails (e.g. permission denied), the process exits with code 1 and logs: `"FATAL: Migration failed: <pg error>"`
- [ ] The full `DATABASE_URL` (including credentials) is never logged — password is stripped from any error message
- [ ] If `DATABASE_URL` is a malformed URL, the app exits with code 1 with the pg parse error included

**Priority:** P0 | **Feature Ref:** F1

---

## Epic 2: Notes REST API (F2)

_Implements the complete CRUD REST API for notes as Next.js Route Handlers. All five endpoints return JSON with standard HTTP status codes. The UI depends exclusively on these endpoints._

---

### US-2.1: List All Notes
**As a** developer (Jordan Kim), **I want to** call `GET /api/notes` and receive all persisted notes ordered newest first, **so that** I can verify the API returns correct data and the UI can populate the note list.

**Acceptance Criteria:**
- [ ] `GET /api/notes` returns HTTP 200 with a JSON array
- [ ] Notes are ordered by `created_at` descending (newest first)
- [ ] Each note object includes: `id`, `title` (or `null`), `body`, `created_at`, `updated_at`
- [ ] Returns an empty array `[]` when no notes exist (not 404 or null)
- [ ] Returns HTTP 500 with `{ "error": "Internal server error" }` on database failure

**Priority:** P0 | **Feature Ref:** F2

---

### US-2.2: Create a Note via API
**As a** developer (Jordan Kim), **I want to** `POST /api/notes` with a title and body, **so that** I can verify the create endpoint returns the persisted note with the correct HTTP status and JSON shape.

**Acceptance Criteria:**
- [ ] `POST /api/notes` with `{ title, body }` returns HTTP 201 with the created note object
- [ ] Created note includes auto-generated `id`, `created_at`, and `updated_at`
- [ ] `title` is stored as `NULL` if absent, null, or empty string after trim
- [ ] Missing or empty `body` (after trim) returns HTTP 400 with `{ "error": "body is required" }`
- [ ] Malformed JSON request body returns HTTP 400 with `{ "error": "Invalid JSON body" }`
- [ ] Returns HTTP 500 with `{ "error": "Internal server error" }` on database failure

**Priority:** P0 | **Feature Ref:** F2

---

### US-2.3: Retrieve a Single Note via API
**As a** developer (Jordan Kim), **I want to** call `GET /api/notes/:id` for an existing and a non-existent note ID, **so that** I can confirm correct 200 and 404 responses.

**Acceptance Criteria:**
- [ ] `GET /api/notes/:id` with a valid existing ID returns HTTP 200 with the note JSON object
- [ ] `GET /api/notes/:id` with a non-existent ID returns HTTP 404 with `{ "error": "Note not found" }`
- [ ] A malformed UUID ID that matches no row returns 404 (not 400)
- [ ] Returns HTTP 500 with `{ "error": "Internal server error" }` on database failure

**Priority:** P0 | **Feature Ref:** F2

---

### US-2.4: Update a Note via API
**As a** developer (Jordan Kim), **I want to** call `PUT /api/notes/:id` with updated title and body, **so that** I can verify the endpoint persists changes, bumps `updated_at`, and returns the updated note.

**Acceptance Criteria:**
- [ ] `PUT /api/notes/:id` with `{ title, body }` returns HTTP 200 with the updated note object
- [ ] `updated_at` is bumped to `now()` on every successful update
- [ ] `title` normalisation rules match POST: null/empty stored as `NULL`
- [ ] Missing or empty `body` (after trim) returns HTTP 400 with `{ "error": "body is required" }`
- [ ] Non-existent `id` returns HTTP 404 with `{ "error": "Note not found" }`
- [ ] Malformed JSON body returns HTTP 400 with `{ "error": "Invalid JSON body" }`

**Priority:** P0 | **Feature Ref:** F2

---

### US-2.5: Delete a Note via API
**As a** developer (Jordan Kim), **I want to** call `DELETE /api/notes/:id` for an existing and a non-existent note, **so that** I can confirm the endpoint returns 204 on success and 404 when the note does not exist.

**Acceptance Criteria:**
- [ ] `DELETE /api/notes/:id` with a valid existing ID returns HTTP 204 with no response body
- [ ] The deleted note no longer appears in subsequent `GET /api/notes` responses
- [ ] Non-existent `id` returns HTTP 404 with `{ "error": "Note not found" }`
- [ ] Returns HTTP 500 with `{ "error": "Internal server error" }` on database failure

**Priority:** P0 | **Feature Ref:** F2

---

## Epic 3: Single-Page UI — Compose Box (F3)

_The primary capture surface. Always visible at the top of the page. Allows Alex to write and submit a new note in under 5 seconds from app open._

---

### US-3.1: Compose and Submit a New Note
**As a** knowledge worker (Alex Rivera), **I want to** type a note in the always-visible compose box and click "Add Note" to save it, **so that** I can capture a fleeting thought in under 5 seconds without any navigation or setup.

**Acceptance Criteria:**
- [ ] Page header displays "QuickNotes" as the application title (`<h1>` or equivalent)
- [ ] Compose box is visible at the top of the page on initial load without any navigation
- [ ] Title input is an `<input type="text">` with placeholder `"Title (optional)"` and is not required
- [ ] Body textarea is a `<textarea>` with placeholder `"Write a note…"`
- [ ] Clicking "Add Note" calls `POST /api/notes` with `{ title: trimmedTitle || null, body: trimmedBody }`
- [ ] On success (HTTP 201), the form is cleared and the note list is refreshed
- [ ] Note creation round-trip (POST + UI update) completes in under 500 ms on a local network

**Priority:** P0 | **Feature Ref:** F3

---

### US-3.2: Prevent Empty Note Submission
**As a** knowledge worker (Alex Rivera), **I want to** be prevented from submitting a note with an empty body, **so that** I never accidentally create blank notes.

**Acceptance Criteria:**
- [ ] "Add Note" button has the native HTML `disabled` attribute when `body.trim() === ""`
- [ ] Button becomes enabled as soon as the body field contains at least one non-whitespace character
- [ ] A whitespace-only body (spaces, tabs, newlines) keeps the button disabled
- [ ] Client-side guard prevents submission even if the disabled attribute is bypassed
- [ ] Server also validates and returns HTTP 400 if an empty body reaches the API

**Priority:** P0 | **Feature Ref:** F3

---

### US-3.3: Handle Compose Box Errors Gracefully
**As a** knowledge worker (Alex Rivera), **I want to** see an inline error message if saving a note fails, **so that** I know to retry and do not lose what I typed.

**Acceptance Criteria:**
- [ ] If `POST /api/notes` returns a non-201 response, an inline error message is shown near the form
- [ ] Error message reads: "Failed to save note. Please try again."
- [ ] The form is NOT cleared on error — title and body values are preserved so the user can retry
- [ ] A network error (fetch throws) shows the same inline error message
- [ ] Button returns to its normal enabled/disabled state after the error (not stuck in loading)

**Priority:** P0 | **Feature Ref:** F3

---

### US-3.4: Compose Box Accessibility
**As a** knowledge worker (Alex Rivera), **I want to** navigate and use the compose box with a keyboard, **so that** I can capture notes without relying on a mouse.

**Acceptance Criteria:**
- [ ] Title input has an associated `<label>` (explicit `htmlFor`/`id` pairing or wrapping label)
- [ ] Body textarea has an associated `<label>` (same pairing requirement)
- [ ] All compose box controls (title input, body textarea, "Add Note" button) are reachable via the Tab key
- [ ] "Add Note" button is activatable via Enter or Space when focused
- [ ] Compose box is rendered in the initial server-side HTML (Next.js SSR)

**Priority:** P0 | **Feature Ref:** F3

---

## Epic 4: Single-Page UI — Note List (F4)

_Below the compose box, a live list of all persisted notes displayed newest first. Alex's primary surface for reviewing captured notes._

---

### US-4.1: View All Notes on Page Load
**As a** knowledge worker (Alex Rivera), **I want to** see all my notes displayed immediately when I open the app, **so that** I can review what I've captured without navigating anywhere.

**Acceptance Criteria:**
- [ ] Note list is populated by calling `GET /api/notes` on page load
- [ ] Each note card displays: title (or "Untitled" if null/empty), body text, and a relative timestamp (e.g. "2 minutes ago")
- [ ] Notes are displayed in newest-first order (as returned by the API — no client-side re-sorting)
- [ ] "Untitled" fallback applies for `null`, empty string `""`, or whitespace-only titles — no blank title area is shown
- [ ] Relative timestamp is rendered using a `<time>` element with a machine-readable `dateTime` attribute
- [ ] Page initial load with notes list rendered completes in under 2 seconds on a typical connection

**Priority:** P0 | **Feature Ref:** F4

---

### US-4.2: View Empty State When No Notes Exist
**As a** knowledge worker (Alex Rivera), **I want to** see a friendly message when I have no notes yet, **so that** I understand the list is working and know what to do next.

**Acceptance Criteria:**
- [ ] When `GET /api/notes` returns an empty array, the message "No notes yet — add your first one above." is displayed
- [ ] The empty state message matches exactly (no paraphrasing)
- [ ] The compose box remains fully usable while the empty state is shown
- [ ] When the first note is created, the empty state disappears and the note appears in the list without a page reload
- [ ] If the last note is deleted, the empty state message reappears without a page reload

**Priority:** P0 | **Feature Ref:** F4

---

### US-4.3: Note List Updates Without Page Reload
**As a** knowledge worker (Alex Rivera), **I want to** see the note list update immediately after I create, edit, or delete a note, **so that** my interactions feel instant and I can trust the app reflects my current data.

**Acceptance Criteria:**
- [ ] Creating a note via the compose box triggers a list refresh (re-fetch `GET /api/notes`) without a full page reload
- [ ] The newly created note appears at the top of the list after submission
- [ ] Editing a note updates that card in place without a full page reload
- [ ] Deleting a note removes that card from the list without a full page reload
- [ ] If `GET /api/notes` fails on refresh, the previous list state is retained and an error message is shown: "Failed to load notes. Refresh to try again."

**Priority:** P0 | **Feature Ref:** F4

---

## Epic 5: Single-Page UI — Inline Edit & Delete (F5)

_Each note card exposes Edit and Delete controls. Alex can correct or remove notes without navigating away from the page._

---

### US-5.1: Edit a Note Inline
**As a** knowledge worker (Alex Rivera), **I want to** click "Edit" on a note card and modify its title or body inline, **so that** I can correct a typo or add context without losing my place in the list.

**Acceptance Criteria:**
- [ ] Clicking "Edit" on a note card transitions it to edit mode with title input and body textarea pre-filled with current values
- [ ] "Save" and "Cancel" buttons replace the "Edit" and "Delete" buttons in edit mode
- [ ] "Save" button is disabled when `body.trim() === ""` in edit mode (native HTML `disabled` attribute)
- [ ] Clicking "Save" calls `PUT /api/notes/:id` with `{ title: trimmedTitle || null, body: trimmedBody }`
- [ ] On success (HTTP 200), the card exits edit mode and displays the updated title, body, and timestamp from the API response
- [ ] Only one note card can be in edit mode at a time

**Priority:** P1 | **Feature Ref:** F5

---

### US-5.2: Cancel an Inline Edit
**As a** knowledge worker (Alex Rivera), **I want to** click "Cancel" while editing a note to discard my changes, **so that** the original note content is preserved if I change my mind.

**Acceptance Criteria:**
- [ ] Clicking "Cancel" in edit mode returns the card to read view without making an API call
- [ ] The card displays the exact original title and body (not trimmed or modified intermediate values)
- [ ] No inline error or loading state is triggered by cancelling
- [ ] The "Edit" and "Delete" buttons reappear after cancellation

**Priority:** P1 | **Feature Ref:** F5

---

### US-5.3: Handle Inline Edit Errors
**As a** knowledge worker (Alex Rivera), **I want to** see an error message if saving my edits fails, **so that** I can retry without losing my changes.

**Acceptance Criteria:**
- [ ] If `PUT /api/notes/:id` returns a non-200 response, the card remains in edit mode
- [ ] An inline error message is displayed near the edit form: "Failed to save. Please try again."
- [ ] A network error (fetch throws) shows the same inline error
- [ ] If the note no longer exists (404), the message reads: "Note not found. It may have been deleted."
- [ ] The "Save" and "Cancel" buttons return to normal state after the error

**Priority:** P1 | **Feature Ref:** F5

---

### US-5.4: Delete a Note with Confirmation
**As a** knowledge worker (Alex Rivera), **I want to** click "Delete" on a note and confirm the action before it is removed, **so that** I don't accidentally delete notes I still need.

**Acceptance Criteria:**
- [ ] Clicking "Delete" shows a confirmation prompt: "Are you sure you want to delete this note?"
- [ ] If the user cancels the confirmation, no API call is made and the card remains unchanged
- [ ] If the user confirms, `DELETE /api/notes/:id` is called
- [ ] On success (HTTP 204), the note card is removed from the list
- [ ] If the deleted note was the last one, the empty state message appears
- [ ] If `DELETE` returns an error (non-204 or network failure), the card is restored (if optimistically hidden) and an error message is shown: "Failed to delete note. Please try again."

**Priority:** P1 | **Feature Ref:** F5

---

### US-5.5: Edit and Delete Controls are Keyboard Accessible
**As a** knowledge worker (Alex Rivera), **I want to** reach and activate edit and delete controls using only the keyboard, **so that** I can manage notes without a mouse.

**Acceptance Criteria:**
- [ ] "Edit" and "Delete" buttons on each note card are reachable via the Tab key
- [ ] "Edit" and "Delete" buttons are activatable via Enter or Space when focused
- [ ] In edit mode, "Save" and "Cancel" buttons are Tab-navigable and keyboard-activatable
- [ ] Focus order within a note card is logical (title → body → Save/Cancel or Edit/Delete)

**Priority:** P1 | **Feature Ref:** F5

---

## Story Index

| Story ID | Title | Persona | Priority | Feature Ref |
|---|---|---|---|---|
| US-0.1 | Start the Application | PER-02 Jordan Kim | P0 | F0 |
| US-0.2 | Verify Application Health | PER-02 Jordan Kim | P0 | F0 |
| US-1.1 | Connect to the Database at Startup | PER-02 Jordan Kim | P0 | F1 |
| US-1.2 | Migrate the Database Schema on Startup | PER-02 Jordan Kim | P0 | F1 |
| US-1.3 | Fail Fast on Missing or Invalid Database Configuration | PER-02 Jordan Kim | P0 | F1 |
| US-2.1 | List All Notes | PER-02 Jordan Kim | P0 | F2 |
| US-2.2 | Create a Note via API | PER-02 Jordan Kim | P0 | F2 |
| US-2.3 | Retrieve a Single Note via API | PER-02 Jordan Kim | P0 | F2 |
| US-2.4 | Update a Note via API | PER-02 Jordan Kim | P0 | F2 |
| US-2.5 | Delete a Note via API | PER-02 Jordan Kim | P0 | F2 |
| US-3.1 | Compose and Submit a New Note | PER-01 Alex Rivera | P0 | F3 |
| US-3.2 | Prevent Empty Note Submission | PER-01 Alex Rivera | P0 | F3 |
| US-3.3 | Handle Compose Box Errors Gracefully | PER-01 Alex Rivera | P0 | F3 |
| US-3.4 | Compose Box Accessibility | PER-01 Alex Rivera | P0 | F3 |
| US-4.1 | View All Notes on Page Load | PER-01 Alex Rivera | P0 | F4 |
| US-4.2 | View Empty State When No Notes Exist | PER-01 Alex Rivera | P0 | F4 |
| US-4.3 | Note List Updates Without Page Reload | PER-01 Alex Rivera | P0 | F4 |
| US-5.1 | Edit a Note Inline | PER-01 Alex Rivera | P1 | F5 |
| US-5.2 | Cancel an Inline Edit | PER-01 Alex Rivera | P1 | F5 |
| US-5.3 | Handle Inline Edit Errors | PER-01 Alex Rivera | P1 | F5 |
| US-5.4 | Delete a Note with Confirmation | PER-01 Alex Rivera | P1 | F5 |
| US-5.5 | Edit and Delete Controls are Keyboard Accessible | PER-01 Alex Rivera | P1 | F5 |

---

## Priority Breakdown

| Priority | Count | Stories |
|---|---|---|
| P0 — Critical | 17 | US-0.1, US-0.2, US-1.1, US-1.2, US-1.3, US-2.1, US-2.2, US-2.3, US-2.4, US-2.5, US-3.1, US-3.2, US-3.3, US-3.4, US-4.1, US-4.2, US-4.3 |
| P1 — High | 5 | US-5.1, US-5.2, US-5.3, US-5.4, US-5.5 |
| P2 — Medium | 0 | — |
| P3 — Low | 0 | — |
| **Total** | **22** | |

---

*Document generated: 2026-07-04*  
*Based on: PRD-quicknotes2.md, FRD-quicknotes2.md, PERSONAS-quicknotes2.md*
