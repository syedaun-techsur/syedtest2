# Jobs-to-be-Done Document
# QuickNotes

| Field | Value |
|---|---|
| **Product Name** | QuickNotes |
| **Acronym** | quicknotes2 |
| **Version** | 1.0 |
| **Date** | 2026-07-04 |
| **Related Personas** | PERSONAS-quicknotes2.md (PER-01, PER-02) |
| **Related PRD** | PRD-quicknotes2.md |
| **Derived From** | PRD Sections 2, 5, 6, 7; Persona goals, pain points, top tasks |

---

## JTBD Summary Table

| JTBD-ID | Persona | Job Statement (abbreviated) | Priority |
|---|---|---|---|
| JTBD-01.1 | PER-01 — Alex Rivera (Knowledge Worker) | Get a thought into a persisted note before it vanishes | P0 |
| JTBD-01.2 | PER-01 — Alex Rivera (Knowledge Worker) | Retrieve and review recently captured notes at a glance | P0 |
| JTBD-01.3 | PER-01 — Alex Rivera (Knowledge Worker) | Correct or expand a note after the fact without losing the original | P1 |
| JTBD-02.1 | PER-02 — Jordan Kim (Developer) | Validate that the app is correctly deployed and operationally healthy | P0 |
| JTBD-02.2 | PER-02 — Jordan Kim (Developer) | Verify database connectivity, schema integrity, and fail-fast behaviour | P0 |
| JTBD-02.3 | PER-02 — Jordan Kim (Developer) | Confirm the REST API contract is correct and automatable | P0 |

---

## PER-01: Alex Rivera — Knowledge Worker / Daily Note-Taker

---

### JTBD-01.1: Capture a Thought Before It Vanishes

**Job Statement:**
When a fleeting idea, task, or reference surfaces mid-meeting or mid-reading, I want to write it into a note and save it in under 5 seconds from opening the app, so I can return to what I was doing without losing the thought.

**Current Alternatives:**
- Opens a separate app (Notion, Apple Notes) — loses the thought during the context switch or the 3–4 second load
- Types into a sticky note or text file — not persisted to a safe location; lost on restart or misplaced
- Sends themselves an email or Slack message — adds inbox noise and requires a separate triage step later
- Writes on physical paper — not searchable, not always within reach

**Hiring Criteria:**
- Compose box is immediately visible without any navigation when the browser tab is opened
- Body field auto-focuses or is one click away — no form wizard, no modal
- "Add Note" button submits the note and clears the form in a single action
- The note appears in the list below within 500 ms of submission — no full page reload needed
- Works with keyboard alone (Tab to field, type, Tab to button or Enter to submit)

**Success Measure:** Alex can open the browser tab, type a note, and see it appear in the persisted list in under 5 seconds — measured end-to-end from page open to note card visible in list.

**Related Features:** F3 (Compose Box), F2 (POST /api/notes), F4 (Note List)
**Priority:** P0

---

### JTBD-01.2: Retrieve a Recently Captured Note at a Glance

**Job Statement:**
When I return to the app to act on something I captured earlier, I want to see all my notes ordered newest-first without any searching or navigation, so I can quickly find the thought I recorded and pick up where I left off.

**Current Alternatives:**
- Scrolls through a long Notion database — requires column sorting and filter setup each session
- Searches email or Slack history — slow and cluttered with unrelated messages
- Re-reads an entire text file — no ordering guarantee; the relevant entry could be anywhere
- Relies on memory alone — high cognitive load; frequently misses items

**Hiring Criteria:**
- Note list renders on page load without any interaction — no "Load notes" button to click
- Most recent note appears at the top; ordering is stable and predictable
- Each note card shows title (or "Untitled"), body, and a human-readable relative timestamp ("2 minutes ago", "yesterday")
- Empty state message is shown when no notes exist, so Alex knows the app is working — not broken
- List reflects the current database state after every create, edit, or delete without requiring a page refresh

**Success Measure:** Alex can identify the most recently captured note within 10 seconds of opening the app, with zero navigation steps required.

**Related Features:** F4 (Note List), F1 (DB persistence), F2 (GET /api/notes)
**Priority:** P0

---

### JTBD-01.3: Correct or Retire a Note After the Fact

**Job Statement:**
When a note I captured earlier is incomplete, contains a typo, or has been actioned and is no longer relevant, I want to edit it in place or delete it with minimal friction, so I can keep my note list accurate without accumulating noise.

**Current Alternatives:**
- Deletes and re-types the entire note from scratch — loses the original timestamp and wastes time
- Leaves incorrect or outdated notes in place — list becomes cluttered and untrustworthy over time
- Adds a correction at the bottom of the note body as a manual annotation — awkward and inconsistent
- Switches to a different tool for "managed" notes and uses QuickNotes only for throwaway capture — defeats the purpose

**Hiring Criteria:**
- "Edit" control on each note card is reachable without navigating away from the main page
- Edit mode pre-populates both title and body with current values — no re-typing required
- "Save" commits the edit in place; note card updates immediately without a full page reload
- "Cancel" reverts to read view with no changes — safe to exit edit mode accidentally
- "Delete" shows a confirmation prompt before removing the note — prevents accidental permanent loss
- All controls are reachable via keyboard (Tab-navigable)

**Success Measure:** Alex can edit a note body and see the updated content reflected in the card in under 10 seconds from clicking "Edit" — no page navigation required.

**Related Features:** F5 (Inline Edit & Delete), F2 (PUT /api/notes/:id, DELETE /api/notes/:id)
**Priority:** P1

---

## PER-02: Jordan Kim — Developer / Technical Evaluator

---

### JTBD-02.1: Validate That the App Is Correctly Deployed and Operationally Healthy

**Job Statement:**
When I have cloned the repo, set `DATABASE_URL`, and run `npm run dev`, I want to confirm the app is live and healthy via a machine-readable endpoint, so I can route traffic to it with confidence and establish a reliable operational baseline before validating individual features.

**Current Alternatives:**
- Manually opens the browser and checks if the page loads — not automatable; gives no signal about DB connectivity or process health
- Inspects `npm run dev` terminal output — useful but not standardized; output format varies across Next.js versions
- Polls a known page URL (`/`) and checks HTTP 200 — works but returns HTML, not a structured health signal
- Skips health validation entirely — leads to silent failures reaching production that are hard to diagnose

**Hiring Criteria:**
- `GET /health` returns HTTP 200 with JSON body `{ "status": "ok" }` — no HTML, no redirect
- Response arrives within 100 ms under normal conditions
- Endpoint is available at the same `0.0.0.0:3000` binding as the rest of the app — no separate port or sidecar required
- Health check is curl-compatible: `curl http://localhost:3000/health` produces a parseable JSON response
- App starts cleanly with a single command (`npm run dev`) given only a valid `DATABASE_URL` — no additional manual migration step, no Docker required

**Success Measure:** Jordan can verify app health with a single `curl` call within 30 seconds of `npm run dev` completing, receiving a valid `{ "status": "ok" }` JSON response with HTTP 200.

**Related Features:** F0 (Scaffold & Health Endpoint), F1 (DB connectivity prerequisite)
**Priority:** P0

---

### JTBD-02.2: Verify Database Connectivity, Schema Integrity, and Fail-Fast Behaviour

**Job Statement:**
When the app starts in a new environment, I want to confirm that database connectivity is established, the `notes` table schema is created automatically, and any misconfiguration produces an immediate descriptive error, so I can trust the data layer is correct before onboarding users or integrating CI/CD pipelines.

**Current Alternatives:**
- Manually connects to the database with `psql` and inspects tables after startup — slow; adds a separate verification step not included in standard dev workflow
- Checks app logs for database error messages — unreliable if the app swallows errors silently
- Writes a custom health check that queries the `notes` table — requires extra code not present in the repo out of the box
- Skips DB validation and discovers schema issues at runtime when the first API call fails — late feedback; expensive to diagnose in CI

**Hiring Criteria:**
- App reads `DATABASE_URL` from environment at startup and fails immediately with a descriptive error message if the variable is absent or malformed — no silent hang
- `notes` table with required columns (`id`, `title`, `body`, `created_at`, `updated_at`) is created or confirmed automatically at startup — no manual migration command needed
- Migrations are idempotent: running `npm run dev` multiple times does not produce schema conflicts or duplicate-table errors
- `PIVOTA_DB_MODE` sidecar mode is supported: app connects directly and runs migrations natively without Docker Compose
- App binds to `0.0.0.0:3000` and is reachable in sandbox/preview environments without additional network configuration

**Success Measure:** On a fresh environment with a valid `DATABASE_URL`, Jordan can inspect the `notes` table immediately after startup and confirm the schema is correct — with zero manual migration steps. On an environment with no `DATABASE_URL`, the app exits with a clear error within 5 seconds of starting.

**Related Features:** F1 (Database Schema & Connectivity), F0 (Scaffold prerequisite)
**Priority:** P0

---

### JTBD-02.3: Confirm the REST API Contract Is Correct and Automatable

**Job Statement:**
When I am evaluating the app as a reference implementation or preparing it for extension, I want to exercise all five CRUD endpoints and verify they return correct HTTP status codes, JSON payloads, and error responses, so I can write reliable automated tests and confidently build on top of the API contract.

**Current Alternatives:**
- Tests the UI manually via browser — no visibility into HTTP status codes or raw JSON; insufficient for API contract validation
- Writes ad-hoc `curl` commands from memory — time-consuming; no reusable test suite; misses edge cases
- Relies solely on frontend smoke testing — `POST /api/notes` returning 201 vs 200 is invisible in a browser interaction
- Copies an API spec from another project and assumes the same behaviour — incorrect assumptions lead to integration bugs

**Hiring Criteria:**
- `GET /api/notes` returns HTTP 200 with a JSON array of notes ordered newest first
- `POST /api/notes` returns HTTP 201 with the created note JSON; returns HTTP 400 with `{ "error": "<message>" }` when `body` is empty or missing
- `GET /api/notes/:id` returns HTTP 200 with the note JSON; returns HTTP 404 with `{ "error": "<message>" }` when the ID does not exist
- `PUT /api/notes/:id` returns HTTP 200 with the updated note; bumps `updated_at`; returns HTTP 404 for unknown IDs
- `DELETE /api/notes/:id` returns HTTP 204 or a success JSON; returns HTTP 404 for unknown IDs
- All error responses follow a consistent `{ "error": "<message>" }` JSON envelope — no plain-text errors, no HTML error pages
- Endpoints are testable via curl, Postman, or an automated test runner without any additional headers or auth

**Success Measure:** Jordan can run an automated test suite covering all five CRUD endpoints (create, list, get-by-id, update, delete) plus the 400 and 404 edge cases and achieve 100% pass rate — with no manual intervention required beyond setting `DATABASE_URL`.

**Related Features:** F2 (Notes REST API), F1 (DB prerequisite)
**Priority:** P0

---

## Outcome-to-Feature Traceability

| JTBD-ID | Related Feature(s) | Expected Outcome |
|---|---|---|
| JTBD-01.1 | F3 (Compose Box), F2 (POST /api/notes), F4 (Note List) | Note composed and visible in list in under 5 seconds; no page reload needed |
| JTBD-01.2 | F4 (Note List), F1 (DB persistence), F2 (GET /api/notes) | All persisted notes rendered newest-first on page load; empty state shown when list is empty |
| JTBD-01.3 | F5 (Inline Edit & Delete), F2 (PUT, DELETE /api/notes/:id) | Note updated or deleted in place; card reflects change immediately; no navigation required |
| JTBD-02.1 | F0 (Scaffold & Health Endpoint), F1 (DB prerequisite) | `GET /health` returns HTTP 200 `{ "status": "ok" }` within 100 ms; app starts with single command |
| JTBD-02.2 | F1 (DB Schema & Connectivity), F0 (Scaffold prerequisite) | Schema auto-created at startup; idempotent migrations; fail-fast on missing `DATABASE_URL` |
| JTBD-02.3 | F2 (Notes REST API), F1 (DB prerequisite) | All 5 CRUD endpoints return correct status codes, JSON payloads, and error envelopes; automatable |

---

## NaC Preview

Candidate Natural Acceptance Criteria derived from job success measures. These will be refined into full NaC statements in STORY-MAP.

| JTBD-ID | Outcome | Candidate Natural Acceptance Criteria |
|---|---|---|
| JTBD-01.1 | Note visible in list in under 5 seconds from page open | Given the app is open in a browser tab, when the user types a note body and clicks "Add Note", then the note card appears at the top of the list within 5 seconds and the compose form is cleared |
| JTBD-01.1 | "Add Note" button is inert when body is empty | Given the compose box is open, when the body textarea is empty or contains only whitespace, then the "Add Note" button is disabled and no POST request is made |
| JTBD-01.2 | Notes rendered newest-first on page load | Given at least one note exists in the database, when the page loads, then all notes are displayed ordered by `created_at` descending with no additional user action |
| JTBD-01.2 | Empty state visible when no notes exist | Given no notes exist in the database, when the page loads, then the message "No notes yet — add your first one above." is displayed in the list area |
| JTBD-01.3 | Edited note reflected immediately in card | Given a note is in edit mode with modified body text, when the user clicks "Save", then the note card updates in place with the new content and `updated_at` is bumped |
| JTBD-01.3 | Delete requires confirmation before removing | Given a note card is visible, when the user clicks "Delete", then a confirmation prompt appears; on confirm, the card is removed from the list; on cancel, the card remains |
| JTBD-02.1 | Health endpoint returns correct JSON within 100 ms | Given the app is running, when `GET /health` is called, then the response is HTTP 200 with body `{ "status": "ok" }` and the response time is under 100 ms |
| JTBD-02.1 | App starts with a single command, no manual steps | Given `DATABASE_URL` is set in the environment, when `npm run dev` is executed, then the app is reachable at `http://localhost:3000` within 30 seconds with no additional commands required |
| JTBD-02.2 | Schema auto-created on first startup | Given an empty PostgreSQL database and a valid `DATABASE_URL`, when `npm run dev` is executed, then the `notes` table with all required columns is present without any manual migration step |
| JTBD-02.2 | Migrations are idempotent across restarts | Given the app has been started and stopped multiple times, when `npm run dev` is executed again, then no schema errors or duplicate-table errors appear in startup logs |
| JTBD-02.2 | Fail-fast on missing DATABASE_URL | Given `DATABASE_URL` is not set, when `npm run dev` is executed, then the app exits within 5 seconds with a descriptive error message indicating the missing variable |
| JTBD-02.3 | POST returns 201 with created note | Given the app is running, when `POST /api/notes` is called with a valid `{ body }` payload, then the response is HTTP 201 with JSON containing the created note including `id` and `created_at` |
| JTBD-02.3 | POST returns 400 for empty body | Given the app is running, when `POST /api/notes` is called with an empty or missing `body`, then the response is HTTP 400 with JSON `{ "error": "<message>" }` |
| JTBD-02.3 | GET/PUT/DELETE return 404 for unknown IDs | Given the app is running, when any of `GET`, `PUT`, or `DELETE /api/notes/:id` is called with a non-existent ID, then the response is HTTP 404 with JSON `{ "error": "<message>" }` |

---

*Document generated: 2026-07-04*
*Derived from: PERSONAS-quicknotes2.md, PRD-quicknotes2.md, .planning/PROJECT.md*
*Next documents: FRD-quicknotes2.md, STORY-MAP-quicknotes2.md*
