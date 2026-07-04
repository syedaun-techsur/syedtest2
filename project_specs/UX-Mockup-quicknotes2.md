# UX Mockup — QuickNotes
# 00: Overview & Design Principles

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Generated:** 2026-07-04  
**Based on:** UserStories-quicknotes2.md, PRD-quicknotes2.md, FRD-quicknotes2.md, JOURNEYS-quicknotes2.md  

---

## UX Approach

QuickNotes is a **single-page, zero-navigation** note-taking application. Every interaction — composing, viewing, editing, deleting — happens on the single root page (`/`). There are no detail pages, no edit routes, no modals that block the full screen. This is a deliberate product constraint derived from persona research (JRN-01.1, JRN-01.3, CP-03): Alex Rivera cannot afford to navigate away from the main page during a meeting.

The UX is organised into two permanent vertical zones:

1. **Top zone — Compose Box (F3):** Always visible above the fold. Captures new notes.
2. **Bottom zone — Note List (F4/F5):** All persisted notes, newest first. Each card exposes inline edit and delete.

---

## Design Principles

### P1: Speed Over Polish
A note must be capturable in under 5 seconds from app open (PRD §7, US-3.1). Every UX decision that adds latency, friction, or navigation is rejected. The compose box body textarea is the primary field; title is secondary.

### P2: Immediate Feedback at Every State Change
Every action produces an unambiguous, immediate signal — button loading state, card update in place, empty-state appearance/disappearance. Silence after an action is a failure state (JRN cross-pattern CP-01).

### P3: Fail Loudly, Never Silently
Inline error messages appear near the action point when API calls fail. Forms are preserved (not cleared) on error. The user always has a clear path to retry (US-3.3, US-5.3, US-5.4).

### P4: No Navigation Required
All CRUD flows execute on the single root page. No `/notes/:id`, no `/notes/new`, no modals requiring route changes (CP-03).

### P5: Accessibility is Structural, Not Additive
Labels, keyboard navigation, ARIA roles, and `<time>` elements are required from initial implementation (US-3.4, US-5.5). Hover-only controls are explicitly rejected (JRN-01.3 Select stage).

---

## Page Structure (High-Level)

```
┌─────────────────────────────────────────────────────────┐
│  HEADER: "QuickNotes"  <h1>                             │
├─────────────────────────────────────────────────────────┤
│  COMPOSE BOX  (always visible, always above the fold)   │
│  ┌───────────────────────────────────────────────────┐  │
│  │  [Title input — optional]                         │  │
│  │  [Body textarea — required, primary focus]        │  │
│  │  [Add Note button — disabled when body empty]     │  │
│  │  [Inline error zone — appears on failure]         │  │
│  └───────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────┤
│  NOTE LIST  (scrollable, newest first)                  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Note Card (read view)                            │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  Title (or "Untitled") | Timestamp           │  │  │
│  │  │  Body text                                   │  │  │
│  │  │  [Edit] [Delete]                             │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  Note Card (edit mode)                            │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  [Title input — pre-filled]                  │  │  │
│  │  │  [Body textarea — pre-filled]                │  │  │
│  │  │  [Save] [Cancel]                             │  │  │
│  │  │  [Inline error zone — appears on failure]    │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  ...more cards...                                 │  │
│  └───────────────────────────────────────────────────┘  │
│  OR                                                     │
│  ┌───────────────────────────────────────────────────┐  │
│  │  "No notes yet — add your first one above."       │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## Persona Summary

| Persona | Primary UX Concern | Key Flows |
|---|---|---|
| PER-01 Alex Rivera (Knowledge Worker) | Speed of capture; trust that notes are saved | Flow-00 (Capture), Flow-01 (Review), Flow-02 (Edit), Flow-03 (Delete) |
| PER-02 Jordan Kim (Developer) | API contract correctness; fail-fast startup | Flow-04 (Health + API verification) — primarily non-UI |

---

## Story Coverage Map

| User Story | UX Section |
|---|---|
| US-3.1 Compose and Submit a New Note | Screen-00 Compose Box, Flow-00 |
| US-3.2 Prevent Empty Note Submission | Screen-00 Compose Box (disabled state) |
| US-3.3 Handle Compose Box Errors | Screen-00 Compose Box (error state) |
| US-3.4 Compose Box Accessibility | Y2-accessibility.md |
| US-4.1 View All Notes on Page Load | Screen-01 Note List, Flow-01 |
| US-4.2 View Empty State | Screen-01 Note List (empty state) |
| US-4.3 Note List Updates Without Reload | Flow-00, Flow-02, Flow-03 |
| US-5.1 Edit a Note Inline | Screen-02 Note Card Edit, Flow-02 |
| US-5.2 Cancel an Inline Edit | Screen-02 Note Card Edit (cancel) |
| US-5.3 Handle Inline Edit Errors | Screen-02 Note Card Edit (error state) |
| US-5.4 Delete a Note with Confirmation | Screen-03 Delete Confirmation, Flow-03 |
| US-5.5 Edit/Delete Keyboard Accessible | Y2-accessibility.md |
| US-0.1, US-0.2 | Flow-04 (developer, non-UI) |
| US-1.1–1.3 | Flow-04 (developer, non-UI) |
| US-2.1–2.5 | API layer, supports all UI flows |
# Flow-00: Capture a Note

**User Story:** US-3.1, US-3.2, US-3.3, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.1 — Capturing a Thought Mid-Meeting  
**Trigger:** User opens or switches to the QuickNotes browser tab  
**Exit:** New note card appears at top of the list; form is cleared  
**Performance target:** POST + UI update < 500 ms (PRD §6)

---

## Flow Diagram

```
[User opens QuickNotes tab]
         │
         ▼
[Page loads — Compose Box visible above fold]
         │
         ▼
[User clicks into Body textarea (or auto-focused)]
         │
         ▼
[User types note body]
         │
         ├──── body.trim() === "" ──▶ [Add Note button stays DISABLED]
         │                                      │
         │                                      ▼
         │                             [User continues typing]
         │
         ├──── body.trim() ≠ "" ────▶ [Add Note button becomes ENABLED]
         │
         ▼
[User optionally types Title]
         │
         ▼
[User clicks "Add Note" (or keyboard submit)]
         │
         ▼
[Button → LOADING state ("Saving…"), temporarily disabled]
         │
         ▼
[POST /api/notes → { title: trimmed|null, body: trimmed }]
         │
         ├── HTTP 201 ──────────────▶ [Clear form fields]
         │                                      │
         │                                      ▼
         │                           [Re-fetch GET /api/notes]
         │                                      │
         │                                      ▼
         │                           [New note card appears at top of list]
         │                                      │
         │                                      ▼
         │                           [Button returns to DISABLED (body now empty)]
         │                                      │
         │                                      ▼
         │                                  [EXIT — Success]
         │
         └── Non-201 / network error ▶ [Show inline error near form]
                                                │
                                                ▼
                                   "Failed to save note. Please try again."
                                                │
                                                ▼
                                   [Form NOT cleared — values preserved]
                                                │
                                                ▼
                                   [Button returns to enabled/disabled per body]
                                                │
                                                ▼
                                          [User retries]
```

---

## Steps Detail

### Step 1: Page Load
- Compose box is rendered in SSR HTML (Next.js server-side render, US-3.4)
- Body textarea receives focus (auto-focus optional; Tab navigation required)
- "Add Note" button renders with `disabled` attribute (body is empty on load, US-3.2)

### Step 2: User Input
- User types in Body textarea (primary field)
- As soon as `body.trim() !== ""`, the `disabled` attribute is removed from the button
- Title input is visible above body but not required; user may skip it
- Whitespace-only body keeps button disabled (US-3.2)

### Step 3: Submit
- User clicks "Add Note" or activates via keyboard (Enter/Space, US-3.4)
- Button shows loading state ("Saving…") and is temporarily disabled to prevent double-submit
- Client sends: `POST /api/notes` with `{ title: titleValue.trim() || null, body: bodyValue.trim() }`

### Step 4a: Success (HTTP 201)
- Both title input and body textarea are cleared to `""`
- Button returns to disabled state (empty body)
- `GET /api/notes` is re-fetched
- New note card appears at top of the list without page reload (US-4.3)

### Step 4b: Error (non-201 or network failure)
- Inline error message appears near the compose box: **"Failed to save note. Please try again."** (US-3.3)
- Form values are preserved (not cleared) so user can retry
- Button returns to normal enabled/disabled state (not stuck in loading)

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Submit | POST | `/api/notes` | 201 + note JSON | Show inline error |
| After success | GET | `/api/notes` | 200 + note array | Retain old list; optional silent retry |
# Flow-01: Review Notes on Page Load

**User Story:** US-4.1, US-4.2, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.2 — Returning to Review and Act on Captured Notes  
**Trigger:** User opens `http://localhost:3000` or switches back to the QuickNotes browser tab  
**Exit:** User has scanned the note list and identified the target note  
**Performance target:** Page initial load with notes list rendered < 2 s (PRD §6)

---

## Flow Diagram

```
[User navigates to http://localhost:3000]
         │
         ▼
[SSR: Page HTML delivered with Compose Box rendered]
         │
         ▼
[Client hydrates — React takes over]
         │
         ▼
[GET /api/notes called on mount]
         │
         ├── Fetching ──────────────▶ [Note list area shows loading indicator]
         │                            (e.g. "Loading notes…" or skeleton cards)
         │
         ├── HTTP 200 — empty [] ──▶ [Empty state message displayed]
         │                            "No notes yet — add your first one above."
         │                                      │
         │                                      ▼
         │                           [Compose box remains fully usable]
         │                                      │
         │                                      ▼
         │                           [User creates first note → Flow-00]
         │                           [Empty state disappears; card appears]
         │
         ├── HTTP 200 — with notes ▶ [Render note cards, newest first]
         │                                      │
         │                                      ▼
         │                           [Each card shows: title or "Untitled",
         │                            body text, relative timestamp]
         │                                      │
         │                                      ▼
         │                           [User scans list — identifies target note]
         │                                      │
         │                                      ▼
         │                                  [EXIT — Flow-02 (Edit)
         │                                   or Flow-03 (Delete)]
         │
         └── HTTP 500 / network ────▶ [Error message in note list area]
                                       "Failed to load notes. Refresh to try again."
                                                │
                                                ▼
                                   [Compose box still fully usable]
                                   [User may manually refresh page]
```

---

## Steps Detail

### Step 1: Server-Side Render
- Next.js delivers the full page HTML including the Compose Box (US-3.4 SSR requirement)
- The note list area is rendered with a loading placeholder or left empty pending client fetch

### Step 2: Client Mount — Fetch Notes
- On React hydration, `GET /api/notes` is called immediately
- A loading indicator is displayed in the list area during the fetch

### Step 3a: Empty State
- If `GET /api/notes` returns `[]`, display: **"No notes yet — add your first one above."** (US-4.2)
- Exact message match required per acceptance criteria
- Compose box remains fully usable
- When user creates the first note (Flow-00), the empty state disappears and the new card appears without page reload (US-4.2)
- If last note is deleted (Flow-03), empty state reappears without page reload (US-4.2)

### Step 3b: Notes List Rendered
- One note card per note, in the API-returned order (newest first — no client re-sorting, US-4.1)
- Each card displays:
  - **Title:** `note.title` if non-null and non-empty; otherwise **"Untitled"** (visually distinguished — italic or muted colour, US-4.1)
  - **Body:** Plain text, no Markdown rendering
  - **Timestamp:** Relative time string (e.g. "2 minutes ago", "yesterday") rendered in a `<time>` element with `dateTime={note.created_at}` attribute (US-4.1)
  - **Controls:** "Edit" button and "Delete" button (F5, US-5.5)

### Step 4: Error State
- If `GET /api/notes` returns 500 or fetch throws, display: **"Failed to load notes. Refresh to try again."** (US-4.3)
- Compose box is still functional
- Previous list state is retained on refresh failure (US-4.3)

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Page load | GET | `/api/notes` | 200 + note array | Show error message in list area |

---

## Information Hierarchy on Notes List

| Priority | Content | Why |
|---|---|---|
| Primary | Newest note (index 0) | Most likely to be the note the user just created or needs |
| Secondary | Relative timestamp | Alex's primary navigation aid (CP-04 from JOURNEYS) |
| Secondary | Body text (first ~2 lines visible) | Identification when title is "Untitled" |
| Tertiary | Title (or "Untitled" fallback) | Secondary identifier |
| Tertiary | Edit / Delete controls | On-demand actions |
# Flow-02: Edit a Note Inline

**User Story:** US-5.1, US-5.2, US-5.3, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.3 — Correcting and Retiring Notes After Action  
**Trigger:** User clicks (or keyboard-activates) the "Edit" button on a note card  
**Exit (success):** Card returns to read view showing updated content  
**Exit (cancel):** Card returns to read view with original content unchanged  
**Performance target:** Card update in place < 500 ms after Save (JRN-01.3)

---

## Flow Diagram

```
[Note card in READ VIEW]
[Shows: title, body, timestamp, "Edit" button, "Delete" button]
         │
         ▼
[User clicks/activates "Edit"]
         │
         ▼
[Card transitions to EDIT MODE]
[Title input — pre-filled with note.title (or "" if null)]
[Body textarea — pre-filled with note.body]
["Save" and "Cancel" replace "Edit" and "Delete"]
[All other cards remain in READ VIEW — only one edit at a time]
         │
         ▼
[User modifies title and/or body]
         │
         ├──── body.trim() === "" ──▶ ["Save" button stays DISABLED]
         │
         ├──── body.trim() ≠ "" ────▶ ["Save" button ENABLED]
         │
         ▼
[User clicks "Save"]                    OR    [User clicks "Cancel"]
         │                                              │
         ▼                                              ▼
[Save button → LOADING ("Saving…")]          [Card reverts to READ VIEW]
[Both buttons temporarily disabled]          [Original values restored]
         │                                   [No API call made]
         ▼                                   [EXIT — Cancel]
[PUT /api/notes/:id → { title, body }]
         │
         ├── HTTP 200 ──────────────▶ [Card exits EDIT MODE]
         │                            [Card displays updated title, body,
         │                             updated_at from API response]
         │                            [No full list re-fetch needed]
         │                                      │
         │                                      ▼
         │                                  [EXIT — Success]
         │
         ├── HTTP 404 ─────────────▶ [Card stays in EDIT MODE]
         │                            Inline error:
         │                            "Note not found. It may have been deleted."
         │                            ["Save" and "Cancel" return to normal]
         │
         └── HTTP 400/500 / network ▶ [Card stays in EDIT MODE]
                                       Inline error:
                                       "Failed to save. Please try again."
                                       ["Save" and "Cancel" return to normal]
                                       [User retries or cancels]
```

---

## Steps Detail

### Step 1: Enter Edit Mode
- User clicks "Edit" button (visible on card in read view, reachable via Tab key, US-5.5)
- Card transitions **in place** — the note card expands/transforms to show inputs, no page navigation
- Title `<input type="text">` pre-filled with `note.title` or `""` if null (US-5.1)
- Body `<textarea>` pre-filled with `note.body` (US-5.1)
- "Save" and "Cancel" buttons replace "Edit" and "Delete" (US-5.1)
- All other cards remain in read view — only one card can be in edit mode at a time (US-5.1)

### Step 2: User Edits
- User modifies title and/or body freely
- "Save" button is disabled when `body.trim() === ""` (native `disabled` attribute, US-5.1)
- User may Tab between title input → body textarea → Save → Cancel (US-5.5)

### Step 3a: Save
- User clicks "Save" or activates via keyboard
- "Save" button shows loading state; both Save and Cancel temporarily disabled (prevents double-submit)
- Client sends: `PUT /api/notes/:id` with `{ title: trimmedTitle || null, body: trimmedBody }`

### Step 4a: Save Success (HTTP 200)
- Card exits edit mode and returns to read view
- Card displays values from the API response (updated title, body, `updated_at`)
- No full `GET /api/notes` re-fetch required — card state updated locally from PUT response (US-5.1)

### Step 4b: Save Error — 404
- Card stays in edit mode (US-5.3)
- Inline error: **"Note not found. It may have been deleted."**
- Save and Cancel buttons return to normal state

### Step 4c: Save Error — 400/500/network
- Card stays in edit mode (US-5.3)
- Inline error: **"Failed to save. Please try again."**
- Save and Cancel buttons return to normal state
- User retries or cancels

### Step 3b: Cancel
- User clicks "Cancel" at any point during edit mode (US-5.2)
- Card reverts to read view immediately
- Original title and body displayed (exact original values, not trimmed/modified intermediates, US-5.2)
- No API call is made
- "Edit" and "Delete" buttons reappear (US-5.2)

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Save | PUT | `/api/notes/:id` | 200 + updated note JSON | Show inline error; stay in edit mode |
# Flow-03: Delete a Note with Confirmation

**User Story:** US-5.4, US-4.2, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.3 — Delete stage  
**Trigger:** User clicks (or keyboard-activates) the "Delete" button on a note card  
**Exit (confirmed):** Note card removed from list; empty state appears if last note  
**Exit (cancelled):** Card unchanged in read view  

---

## Flow Diagram

```
[Note card in READ VIEW]
[Shows: title, body, timestamp, "Edit" button, "Delete" button]
         │
         ▼
[User clicks/activates "Delete"]
         │
         ▼
[Confirmation prompt shown]
"Are you sure you want to delete this note?"
[OK / Cancel]
         │
         ├── User clicks CANCEL ───▶ [No API call]
         │                            [Card unchanged in READ VIEW]
         │                            [EXIT — Cancelled]
         │
         └── User clicks OK ────────▶ [Optional: card optimistically hidden]
                                                │
                                                ▼
                                   [DELETE /api/notes/:id]
                                                │
                                   ┌────────────┴─────────────┐
                                   │                          │
                              HTTP 204                  Non-204 / network
                                   │                          │
                                   ▼                          ▼
                          [Card permanently         [Restore card if
                           removed from list]        optimistically hidden]
                                   │                          │
                                   ▼                          ▼
                          [If last note:             Inline / toast error:
                           empty state appears]   "Failed to delete note.
                                   │               Please try again."
                                   ▼
                               [EXIT — Success]
```

---

## Steps Detail

### Step 1: User Clicks Delete
- "Delete" button is visible on note card in read view
- Reachable via Tab key; activatable via Enter or Space (US-5.5)
- Button is always visible (not hover-only — JRN-01.3 Select stage pain point)

### Step 2: Confirmation Prompt
- A confirmation prompt appears: **"Are you sure you want to delete this note?"** (US-5.4)
- Implementation: `window.confirm()` is acceptable for v1 (FRD F05)
- The prompt is synchronous — it blocks interaction until dismissed
- **If user cancels:** No API call is made; card remains in read view (US-5.4)
- **If user confirms:** Proceed to Step 3

### Step 3: Delete Request
- Optionally, the card is **optimistically hidden** from the list immediately after confirmation, before the API responds — for perceived responsiveness (FRD F05)
- Client sends: `DELETE /api/notes/:id`

### Step 4a: Success (HTTP 204)
- Card is permanently removed from the list (US-5.4)
- If the deleted note was the last one, the empty state message appears: **"No notes yet — add your first one above."** (US-4.2, US-5.4)
- No page reload required (US-4.3)

### Step 4b: Error (non-204 or network failure)
- If card was optimistically hidden, it is **restored** to the list (US-5.4)
- Inline or toast error message: **"Failed to delete note. Please try again."** (US-5.4)
- Card remains in read view; user may retry

### Edge Case: 404 on Delete
- If `DELETE` returns 404 (note already gone from DB), card is removed from list anyway
- Rationale: The desired state (note not in list) is already achieved (FRD F05 error table)

---

## Confirmation Prompt Design Note

The confirmation is a single-step prompt (`window.confirm`). It is:
- **Present but not disruptive** — does not block the full page with a custom modal (JRN-01.3 Delete stage)
- **Not a two-page flow** — no route navigation
- A custom inline confirmation widget is acceptable as a v1.1 enhancement if `window.confirm` is considered too jarring

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Confirmed delete | DELETE | `/api/notes/:id` | 204 No Content | Restore card; show error message |
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
# Screen-00: Compose Box

**Purpose:** Primary capture surface — always visible at the top of the page. Allows users to write and submit a new note in under 5 seconds.  
**User Stories:** US-3.1, US-3.2, US-3.3, US-3.4  
**Feature:** F3  
**Location:** Top zone of the single root page (`/`), above the fold on all viewports

---

## Layout

```
┌─────────────────────────────────────────────────────────────┐
│  <h1> QuickNotes </h1>                                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── COMPOSE BOX ───────────────────────────────────────┐  │
│  │                                                       │  │
│  │  <label for="title">Title</label>                     │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  Title (optional)                               │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │  <input type="text" id="title" placeholder="Title     │  │
│  │   (optional)" — not required>                         │  │
│  │                                                       │  │
│  │  <label for="body">Note</label>                       │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │                                                 │  │  │
│  │  │  Write a note…                                  │  │  │
│  │  │                                                 │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │  <textarea id="body" placeholder="Write a note…">     │  │
│  │                                                       │  │
│  │  [ Add Note ]  ← <button disabled> when body empty    │  │
│  │                                                       │  │
│  │  (error zone — hidden by default)                     │  │
│  │  ⚠ Failed to save note. Please try again.             │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  NOTE LIST (see Screen-01)                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Information Hierarchy

| Priority | Content | Placement | Notes |
|---|---|---|---|
| Primary | Body textarea | Centre, large touch target | Required field; receives focus first |
| Secondary | "Add Note" button | Below body textarea | Disabled state when body empty |
| Tertiary | Title input | Above body textarea | Optional; not required |
| Contextual | Inline error message | Below button | Only visible on failure |
| Page-level | "QuickNotes" `<h1>` | Page header | Identifies app; not repeated |

---

## Interactive Elements

| Element | Type | Behaviour |
|---|---|---|
| Title input | `<input type="text">` | Optional; placeholder "Title (optional)"; associated `<label htmlFor="title">` |
| Body textarea | `<textarea>` | Required; placeholder "Write a note…"; associated `<label htmlFor="body">`; drives button disabled state |
| Add Note button | `<button>` | Disabled (native `disabled` attr) when `body.trim() === ""`; enabled when body has content |
| Inline error zone | `<p>` or `<div>` | Hidden by default; shown on POST failure; message: "Failed to save note. Please try again." |

---

## States

| State | Visual Appearance | User Feedback |
|---|---|---|
| **Default (empty)** | Title input empty, body textarea empty, "Add Note" button disabled (greyed out) | Button visually de-emphasised; not clickable |
| **Body has content** | Body textarea has text, "Add Note" button enabled (normal appearance) | Button becomes interactive |
| **Loading / Saving** | "Add Note" button shows "Saving…" text, button disabled temporarily | Prevents double-submit; clear in-progress signal |
| **Success** | Both fields cleared, button returns to disabled | Note appears in list below — visual confirmation |
| **Error** | Fields preserved with user's input; red/warning inline message below button | "Failed to save note. Please try again." |

---

## Validation Rules

| Rule | Client Behaviour | Server Fallback |
|---|---|---|
| Body required | Button has native `disabled` attribute; also guarded client-side before fetch | API returns 400 `{ "error": "body is required" }` |
| Whitespace-only body | `body.trim() === ""` → button stays disabled | Same 400 response |
| Title optional | No restriction; empty string sent as `null` to API | Stored as NULL |
| Form not cleared on error | Values preserved for retry | N/A |

---

## DOM / Accessibility Requirements

```html
<!-- Required structure (US-3.4) -->
<form>
  <label for="title">Title</label>
  <input type="text" id="title" placeholder="Title (optional)" />

  <label for="body">Note</label>
  <textarea id="body" placeholder="Write a note…"></textarea>

  <button type="submit" disabled>Add Note</button>

  <!-- Error zone — conditionally rendered -->
  <p role="alert" aria-live="polite">Failed to save note. Please try again.</p>
</form>
```

- Tab order: Title input → Body textarea → Add Note button (US-3.4)
- "Add Note" activatable via Enter or Space when focused (US-3.4)
- Compose box in SSR initial HTML (not client-render-only, US-3.4)
- Error zone uses `role="alert"` or `aria-live="polite"` so screen readers announce it
# Screen-01: Note List

**Purpose:** Display all persisted notes, newest first. Primary read surface for reviewing captured content.  
**User Stories:** US-4.1, US-4.2, US-4.3  
**Feature:** F4  
**Location:** Below the Compose Box on the single root page (`/`); scrollable

---

## Layout — Non-Empty State

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (Screen-00 — above)                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── NOTE LIST ─────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  ┌─── NOTE CARD (newest) ──────────────────────────┐  │  │
│  │  │  My Meeting Note        2 minutes ago            │  │  │
│  │  │  Follow up with Sam re: pricing by end of week.  │  │  │
│  │  │                          [Edit]  [Delete]        │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │                                                       │  │
│  │  ┌─── NOTE CARD ───────────────────────────────────┐  │  │
│  │  │  Untitled                    yesterday            │  │  │
│  │  │  Check contract — which one?                      │  │  │
│  │  │                          [Edit]  [Delete]        │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │                                                       │  │
│  │  ┌─── NOTE CARD ───────────────────────────────────┐  │  │
│  │  │  Project Kickoff             3 days ago           │  │  │
│  │  │  Action items from kickoff: 1) Set up repo…      │  │  │
│  │  │                          [Edit]  [Delete]        │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │                                                       │  │
│  │  ... more cards (scroll) ...                          │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Layout — Empty State

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (Screen-00 — above, fully usable)              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── NOTE LIST ─────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  No notes yet — add your first one above.             │  │
│  │  (exact message required, US-4.2)                     │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Layout — Loading State

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (Screen-00 — above)                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── NOTE LIST ─────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  Loading notes…                                       │  │
│  │  (or skeleton card placeholders)                      │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Layout — Error State

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (Screen-00 — above, still usable)              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── NOTE LIST ─────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  ⚠ Failed to load notes. Refresh to try again.        │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Note Card Anatomy (Read View)

```
┌──────────────────────────────────────────────────────────┐
│  [Title or "Untitled" (italic/muted)]   [Timestamp]      │
│  <time dateTime="2026-07-04T12:00:00Z">2 minutes ago</time>
│                                                          │
│  Body text — plain text, as many lines as needed         │
│  (no Markdown rendering in v1)                           │
│                                                          │
│                              [Edit]  [Delete]            │
└──────────────────────────────────────────────────────────┘
```

### Card Fields

| Field | Source | Fallback | Notes |
|---|---|---|---|
| Title | `note.title` | **"Untitled"** (italic or muted colour) | Null, `""`, whitespace → "Untitled"; no blank title area |
| Timestamp | `note.created_at` | — | Relative: "just now", "2 minutes ago", "yesterday" |
| `<time>` element | `dateTime={note.created_at}` | — | Machine-readable ISO 8601 UTC for screen readers (US-4.1) |
| Body | `note.body` | — | Plain text; never empty (API ensures this) |
| Edit button | — | — | Triggers Flow-02; keyboard accessible |
| Delete button | — | — | Triggers Flow-03; keyboard accessible |

---

## Information Hierarchy

| Priority | Content | Placement | Notes |
|---|---|---|---|
| Primary | Body text | Card body | Content; may be multi-line |
| Primary | Relative timestamp | Card top-right | Navigation aid (CP-04 — timestamps are functional) |
| Secondary | Title or "Untitled" | Card top-left | Identifier |
| Tertiary | Edit / Delete controls | Card bottom-right | On-demand actions; always visible |

---

## States

| State | Appearance | Trigger |
|---|---|---|
| **Loading** | "Loading notes…" or skeleton placeholders | Initial page load, before GET /api/notes resolves |
| **Empty** | "No notes yet — add your first one above." | GET /api/notes returns `[]` |
| **Populated** | List of note cards, newest at top | GET /api/notes returns non-empty array |
| **Error** | "Failed to load notes. Refresh to try again." | GET /api/notes returns 5xx or fetch throws |
| **Post-create update** | New card appears at top; empty state disappears | After successful POST (Flow-00) |
| **Post-delete update** | Card removed from list; empty state appears if last | After successful DELETE (Flow-03) |
| **Post-edit update** | Card content updated in place | After successful PUT (Flow-02) |

---

## List Behaviour Rules

- **Order:** Newest first, exactly as returned by API (`ORDER BY created_at DESC`). No client-side re-sorting. (US-4.1)
- **Live updates:** List updates without full page reload after any create, edit, or delete (US-4.3)
- **Refresh failure:** If GET /api/notes fails on a background refresh (after create/edit/delete), the previous list state is retained and an error message is shown (US-4.3)
- **Only one card in edit mode at a time** (US-5.1) — all other cards stay in read view
# Screen-02: Note Card — Edit Mode

**Purpose:** Inline editing of a single note card, in place within the note list. No page navigation.  
**User Stories:** US-5.1, US-5.2, US-5.3, US-5.5  
**Feature:** F5  
**Location:** Within the Note List (Screen-01), replacing the read view of the targeted card

---

## Layout — Edit Mode (Card Expanded)

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (above — unchanged, still usable)              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── NOTE CARD (EDIT MODE) ──────────────────────────────┐ │
│  │                                                        │ │
│  │  <label for="edit-title-{id}">Title</label>            │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │  My Meeting Note                                 │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │  <input type="text" id="edit-title-{id}"               │ │
│  │   value="My Meeting Note">                             │ │
│  │                                                        │ │
│  │  <label for="edit-body-{id}">Note</label>              │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │                                                  │  │ │
│  │  │  Follow up with Sam re: pricing by end of week.  │  │ │
│  │  │                                                  │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │  <textarea id="edit-body-{id}">                        │ │
│  │                                                        │ │
│  │  [ Save ]  [ Cancel ]                                  │ │
│  │  (Save disabled when body empty)                       │ │
│  │                                                        │ │
│  │  (error zone — hidden by default)                      │ │
│  │  ⚠ Failed to save. Please try again.                   │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌─── OTHER NOTE CARDS (read view, unchanged) ────────────┐ │
│  │  ...                                                   │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Read View vs Edit Mode Comparison

| Element | Read View | Edit Mode |
|---|---|---|
| Title display | Plain text (or "Untitled" italic) | `<input type="text">` pre-filled with `note.title` |
| Body display | Plain text | `<textarea>` pre-filled with `note.body` |
| Timestamp | `<time>` relative string | Hidden during edit mode |
| Action buttons | "Edit" + "Delete" | "Save" + "Cancel" |
| Error zone | Not present | Shown on PUT failure |

---

## Interactive Elements (Edit Mode)

| Element | Type | Behaviour |
|---|---|---|
| Title input | `<input type="text" id="edit-title-{id}">` | Pre-filled with `note.title` or `""` if null; optional |
| Body textarea | `<textarea id="edit-body-{id}">` | Pre-filled with `note.body`; required (drives Save disabled state) |
| Save button | `<button>` | Disabled (native `disabled` attr) when `body.trim() === ""`; calls PUT on click |
| Cancel button | `<button>` | Always enabled; reverts card to read view; no API call |
| Inline error zone | `<p role="alert">` | Shown on PUT failure; hidden by default |

---

## States (Edit Mode)

| State | Appearance | User Feedback |
|---|---|---|
| **Initial (just opened)** | Inputs pre-filled; Save enabled if body non-empty | Ready to edit |
| **Body cleared by user** | Save button becomes disabled | Cannot save empty body |
| **Saving** | Save button shows "Saving…"; both buttons disabled | In-progress signal; prevents double-submit |
| **Save success** | Card exits edit mode; read view shows updated content | Immediate in-place update |
| **Save error — generic** | Card stays in edit mode; error message shown | "Failed to save. Please try again." |
| **Save error — 404** | Card stays in edit mode; specific error message | "Note not found. It may have been deleted." |
| **Cancelled** | Card reverts to read view; original values restored | No changes persisted |

---

## Validation Rules (Edit Mode)

| Rule | Behaviour |
|---|---|
| Body must be non-empty after trim | Save button has native `disabled` attribute |
| Title is optional | Empty string normalised to `null` on save |
| Cancel restores exact original values | Do not persist trimmed or modified intermediates |
| Only one card in edit mode at a time | Other cards remain in read view (US-5.1) |

---

## DOM / Accessibility Requirements (Edit Mode)

```html
<!-- Edit mode card structure (US-5.5) -->
<article aria-label="Editing note: {title}">
  <label for="edit-title-{id}">Title</label>
  <input type="text" id="edit-title-{id}" value="{note.title}" />

  <label for="edit-body-{id}">Note</label>
  <textarea id="edit-body-{id}">{note.body}</textarea>

  <button type="submit" disabled="{bodyEmpty}">Save</button>
  <button type="button">Cancel</button>

  <!-- Error zone — conditionally rendered -->
  <p role="alert" aria-live="polite">{errorMessage}</p>
</article>
```

- Tab order within card: Title input → Body textarea → Save → Cancel (US-5.5)
- Save and Cancel activatable via Enter/Space when focused (US-5.5)
- Focus is not trapped — user can Tab out of the card (accessibility best practice)
- `id` attributes on inputs must be unique per card (include note ID in `id`)

---

## Error Messages

| Scenario | Error Message | Card Behaviour |
|---|---|---|
| PUT returns 500 or network failure | "Failed to save. Please try again." | Stays in edit mode |
| PUT returns 404 (note gone) | "Note not found. It may have been deleted." | Stays in edit mode |
| PUT returns 400 (body empty — server guard) | "Failed to save. Please try again." | Stays in edit mode (should not occur if client validates) |
# Screen-03: Delete Confirmation

**Purpose:** Confirm user intent before permanently deleting a note. Prevents accidental data loss.  
**User Stories:** US-5.4, US-4.2  
**Feature:** F5  
**Location:** Triggered from a note card's "Delete" button; appears as a browser dialog or inline prompt

---

## Confirmation Prompt — v1 Implementation (`window.confirm`)

```
┌─────────────────────────────────────────────────────────────┐
│  BROWSER DIALOG (native)                                    │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  Are you sure you want to delete this note?           │  │
│  │                                                       │  │
│  │                    [  OK  ]   [ Cancel ]              │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

- **Prompt text (exact):** "Are you sure you want to delete this note?" (US-5.4)
- **Implementation:** `window.confirm(...)` is acceptable for v1
- **OK:** Proceeds with `DELETE /api/notes/:id`
- **Cancel:** No action; card unchanged in read view

---

## After Confirmation — Visual States

### Deleting (in-progress)
```
┌─────────────────────────────────────────────────────────────┐
│  NOTE LIST                                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  (Card may be optimistically hidden / faded out)     │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Other cards remain visible and interactive          │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Delete Success — Last Note Remaining
```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (unchanged)                                    │
├─────────────────────────────────────────────────────────────┤
│  NOTE LIST                                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  No notes yet — add your first one above.            │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Delete Error (Non-204 or Network Failure)
```
┌─────────────────────────────────────────────────────────────┐
│  NOTE LIST                                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  My Meeting Note           2 minutes ago              │   │
│  │  Follow up with Sam re: pricing by end of week.       │   │
│  │                             [Edit]  [Delete]          │   │
│  └──────────────────────────────────────────────────────┘   │
│  ⚠ Failed to delete note. Please try again.                 │
│  (toast or inline message near the card)                    │
└─────────────────────────────────────────────────────────────┘
```

---

## States

| State | Appearance | Trigger |
|---|---|---|
| **Confirmation pending** | Browser `confirm()` dialog overlays page | User clicks "Delete" |
| **User cancels** | Dialog dismissed; card unchanged in read view | User clicks Cancel in dialog |
| **Deleting (optimistic)** | Card may be faded/hidden; other cards normal | User clicks OK in dialog |
| **Delete success** | Card removed from DOM; empty state if last | `DELETE` returns 204 |
| **Delete error** | Card restored (if hidden); inline error shown | `DELETE` returns non-204 or network fails |

---

## Error Messages

| Scenario | Message | Placement |
|---|---|---|
| DELETE returns non-204 or network error | "Failed to delete note. Please try again." | Inline near card or toast notification |
| DELETE returns 404 (note already gone) | Card removed anyway — desired state achieved | No error shown (silent success) |

---

## Design Rationale

- **Single confirmation step** — one `confirm()` call; not a multi-step modal (JRN-01.3: "not disruptive")
- **No route change** — confirmation happens on the same page (CP-03: No Navigation Required)
- **Optimistic removal** is optional in v1 — implementation may hide the card before API response returns, then restore on error
- **v1.1 opportunity:** Replace `window.confirm()` with an inline confirmation widget (e.g. "Are you sure? [Yes, delete] [Keep]" rendered within or below the card) for a less jarring UX
# Y0: Interaction Patterns

**Applies to:** All screens and flows  
**User Stories:** US-3.1–3.4, US-4.1–4.3, US-5.1–5.5

---

## Pattern 1: Disabled Button as Validation Gate

**When to use:** Any form action where a required field must have content before submission.  
**Used in:** Compose Box "Add Note" button (US-3.2), Note Card "Save" button in edit mode (US-5.1)

**Behaviour:**
- The `<button>` element carries the native HTML `disabled` attribute (not CSS-only)
- The disabled state is driven by `fieldValue.trim() === ""`
- Whitespace-only input keeps the button disabled
- As soon as the user types a non-whitespace character, the button becomes enabled
- The button returns to its correct enabled/disabled state after any error (not stuck in loading)

**Why native `disabled` matters:** CSS-only visual disabling does not prevent keyboard users or assistive technology from activating the button. Native `disabled` prevents all activation paths.

**Example DOM:**
```html
<button type="submit" disabled>Add Note</button>   <!-- body empty -->
<button type="submit">Add Note</button>             <!-- body has content -->
```

---

## Pattern 2: Inline Error Near Action Point

**When to use:** Any API call that can fail, where the user needs to know they should retry.  
**Used in:** Compose Box on POST failure (US-3.3), Note Card on PUT failure (US-5.3), Note List on GET failure (US-4.3), Delete on DELETE failure (US-5.4)

**Behaviour:**
- Error message appears **near the element that triggered the action** (below the button, within the card)
- Error messages are concise and actionable:
  - POST fail: "Failed to save note. Please try again."
  - PUT fail: "Failed to save. Please try again."
  - PUT 404: "Note not found. It may have been deleted."
  - DELETE fail: "Failed to delete note. Please try again."
  - GET fail: "Failed to load notes. Refresh to try again."
- The error zone uses `role="alert"` or `aria-live="polite"` to announce to screen readers
- The error disappears when the user successfully retries or navigates away
- **Form values are preserved** on error — the user should never need to retype their content

**Placement rule:** Error message appears between the last form field and the footer of the component (below the button row).

---

## Pattern 3: Loading State on Submit Actions

**When to use:** Any action that triggers an API call with latency.  
**Used in:** "Add Note" button during POST (Flow-00), "Save" button during PUT (Flow-02)

**Behaviour:**
- Button label changes to "Saving…" (or similar) while awaiting API response
- Button is temporarily `disabled` during the request to prevent double-submission
- Both Save and Cancel buttons are disabled during the Save request in edit mode (prevents cancelling a save mid-flight)
- On completion (success or error), button returns to appropriate state
- **Target:** POST + UI update should complete in < 500 ms on local network, making the loading state brief but present

---

## Pattern 4: In-Place Card Transition (Read ↔ Edit)

**When to use:** Switching a note card between read view and edit mode.  
**Used in:** Note Card edit mode entry/exit (US-5.1, US-5.2)

**Behaviour:**
- The card transitions **in place** within the list — it does not navigate to a new page or open a modal
- In read view: title text, body text, timestamp, "Edit" and "Delete" buttons
- In edit mode: title input, body textarea, "Save" and "Cancel" buttons (replace Edit/Delete)
- Timestamp is hidden during edit mode (not relevant while editing)
- **Only one card** can be in edit mode at a time — opening a new edit (if implemented) auto-cancels the previous
- On Cancel: card reverts to read view instantly, no animation required
- On Save success: card updates to show new values from API response, no full list re-fetch

---

## Pattern 5: Optimistic List Update

**When to use:** After a create, edit, or delete action that succeeds.  
**Used in:** After POST (new card appears, Flow-00), after DELETE (card removed, Flow-03)

**Behaviour:**
- **Create (POST):** After success, re-fetch `GET /api/notes` and re-render the list. The new card appears at the top. No page reload.
- **Edit (PUT):** Update the card content locally from the PUT response body — no full list re-fetch required.
- **Delete (DELETE):** Remove the card from local state immediately on 204. Optionally, hide it optimistically before the API responds and restore on error.
- **Empty state transitions:** When the first note is created, the empty state message disappears immediately. When the last note is deleted, the empty state message appears immediately. Both without page reload (US-4.2).

---

## Pattern 6: Preserve Form State on Error

**When to use:** Any submit action that results in an API error.  
**Used in:** Compose Box on POST failure (US-3.3), Note Card on PUT failure (US-5.3)

**Behaviour:**
- On API error or network failure, **do not clear the form**
- The user's typed content is preserved so they can retry without retyping
- The inline error message appears near the button
- The button returns to its correct enabled/disabled state (not stuck in loading)
- This pattern is intentional — losing user input on error is a high-frustration failure mode (JRN-01.1 Submit stage)

---

## Pattern 7: Confirmation Before Destructive Action

**When to use:** Actions that permanently delete data.  
**Used in:** Delete note (US-5.4, Flow-03)

**Behaviour:**
- User must explicitly confirm before delete executes
- v1: `window.confirm("Are you sure you want to delete this note?")`
- On Cancel: no API call; card unchanged
- On Confirm: DELETE request proceeds
- Single confirmation step — not a two-step wizard (JRN-01.3 Delete stage: "not disruptive")
- v1.1 opportunity: inline confirmation widget within the card

---

## Pattern 8: Timestamp as Navigation Aid

**When to use:** Every note card, always.  
**Used in:** All note cards in read view (US-4.1, JRN-01.2 Scan/Identify stages)

**Behaviour:**
- Every card shows a **relative timestamp** derived from `note.created_at`
- Format: "just now", "2 minutes ago", "1 hour ago", "yesterday", "3 days ago"
- Rendered using: `<time dateTime="{note.created_at}">{relativeString}</time>`
- The `dateTime` attribute contains the ISO 8601 UTC string for machine readability
- Timestamps are Alex's **primary navigation mechanism** when titles are "Untitled" (CP-04)
- Static render on fetch is acceptable for v1; live ticking (auto-update without refetch) is optional
# Y1: Responsive Considerations

**Applies to:** All screens  
**User Stories:** US-3.1, US-3.4, US-4.1, US-5.5  
**Browser targets:** Chrome, Firefox, Safari, Edge (latest evergreen, PRD §6)

---

## Layout Philosophy

QuickNotes is a **single-column, single-page** application. The layout is inherently responsive by nature — there is no complex grid, sidebar, or multi-panel layout to manage. The primary responsive concern is ensuring the compose box remains **above the fold** on all viewports, so Alex can start typing immediately without scrolling.

---

## Breakpoint Definitions

| Breakpoint | Label | Width Range |
|---|---|---|
| Mobile | sm | < 640px |
| Tablet | md | 640px – 1023px |
| Desktop | lg | ≥ 1024px |

---

## Desktop (≥ 1024px)

```
┌────────────────────────────────────────────────────────────────┐
│  QuickNotes                                                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌── COMPOSE BOX (max-width: ~720px, centred) ──────────────┐  │
│  │  Title input (full width of box)                         │  │
│  │  Body textarea (full width, ~4 rows min)                 │  │
│  │  [Add Note] button (right-aligned or left-aligned)       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                │
│  ┌── NOTE LIST (max-width: ~720px, centred) ────────────────┐  │
│  │  ┌── Note Card ────────────────────────────────────────┐ │  │
│  │  │  Title                              Timestamp       │ │  │
│  │  │  Body text (full width of card)                     │ │  │
│  │  │                              [Edit]  [Delete]       │ │  │
│  │  └────────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**Desktop considerations:**
- Page content is constrained to a readable max-width (~720px) and centred horizontally
- Compose box and note list align in a single column
- Body textarea has comfortable height (~4–6 rows minimum) — no forced scroll to type
- "Add Note" button is clearly visible without scrolling on a standard 1080p screen

---

## Tablet (640px – 1023px)

```
┌──────────────────────────────────────────────────────┐
│  QuickNotes                                          │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌── COMPOSE BOX (full width, ~16px padding) ─────┐  │
│  │  Title input                                   │  │
│  │  Body textarea (~3–4 rows)                     │  │
│  │  [Add Note]                                    │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌── NOTE LIST (full width) ──────────────────────┐  │
│  │  ┌── Note Card ────────────────────────────┐   │  │
│  │  │  Title                 Timestamp        │   │  │
│  │  │  Body text                              │   │  │
│  │  │                    [Edit]  [Delete]     │   │  │
│  │  └────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────┘
```

**Tablet considerations:**
- Layout remains single-column; no structural changes from desktop
- Content fills available width with appropriate horizontal padding (~16px per side)
- Compose box must still be fully visible above the fold on a typical 768px portrait tablet
- Touch targets for buttons should be at minimum 44×44px (WCAG 2.5.5)

---

## Mobile (< 640px)

```
┌──────────────────────────────────────────┐
│  QuickNotes                              │
├──────────────────────────────────────────┤
│                                          │
│  ┌── COMPOSE BOX (full width) ─────────┐ │
│  │  Title input (full width)           │ │
│  │  Body textarea (~3 rows)            │ │
│  │  [Add Note]  ← full-width button    │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌── NOTE LIST ────────────────────────┐ │
│  │  ┌── Note Card ──────────────────┐  │ │
│  │  │  Title         Timestamp      │  │ │
│  │  │  Body text                    │  │ │
│  │  │  [Edit]  [Delete]             │  │ │
│  │  └──────────────────────────────┘  │ │
│  └────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

**Mobile considerations:**
- Compose box must be above the fold on a typical 375px-wide mobile viewport
- Body textarea minimum height: ~3 rows (enough to see the placeholder and start typing)
- "Add Note" button: full-width or near-full-width for easy thumb access
- Note card title and timestamp may stack vertically if space is tight (title on top row, timestamp below or right)
- "Edit" and "Delete" buttons must remain clearly visible and tappable (min 44px touch target)
- No hover states required on mobile — controls must be always visible (JRN-01.3: no hover-only controls)

---

## Critical Cross-Breakpoint Rules

| Rule | Reason |
|---|---|
| Compose box is ALWAYS above the fold | JRN-01.1 Orient stage: Alex cannot scroll to find it |
| Edit and Delete buttons are ALWAYS visible (not hover-only) | JRN-01.3: hover-only controls are a pain point; mobile has no hover |
| Touch targets ≥ 44×44px | WCAG 2.5.5 minimum; prevents accidental taps |
| No horizontal scroll | Single-column layout prevents overflow |
| Inline error messages stack below buttons | Not floated or overlaid — must not obscure content |

---

## Styling Approach

Per PRD §4: Tailwind CSS or CSS Modules are acceptable. The layout can be achieved with minimal styling:

```
Suggested Tailwind class pattern:
- Page container: max-w-2xl mx-auto px-4
- Section (compose/list): space-y-4
- Note card: rounded-lg border p-4
- Button: min-h-[44px] px-4 py-2
- Textarea: w-full min-h-[80px] resize-y
```

No complex grid framework required. A simple flexbox column layout suffices.
# Y2: Accessibility Notes

**Applies to:** All screens  
**User Stories:** US-3.4 (Compose Box), US-5.5 (Edit/Delete Controls)  
**Standard:** WCAG 2.1 AA as baseline

---

## Summary of Requirements

QuickNotes has two explicitly specified accessibility requirements as acceptance criteria:
1. **US-3.4** — Compose box fully keyboard-accessible with proper label associations
2. **US-5.5** — Edit and Delete controls on note cards keyboard-accessible

All other accessibility notes below are strongly recommended to meet WCAG 2.1 AA and serve users who rely on assistive technology.

---

## 1. Label Associations (US-3.4, US-5.5)

Every form input must have an associated `<label>` element. Two valid patterns:

**Pattern A — Explicit `htmlFor` / `id` pairing (preferred):**
```html
<label htmlFor="body">Note</label>
<textarea id="body" placeholder="Write a note…"></textarea>
```

**Pattern B — Wrapping label:**
```html
<label>
  Note
  <textarea placeholder="Write a note…"></textarea>
</label>
```

**Required label associations:**

| Control | Label Text | Location |
|---|---|---|
| Compose box title input | "Title" | Compose box (US-3.4) |
| Compose box body textarea | "Note" | Compose box (US-3.4) |
| Edit mode title input | "Title" | Note card edit mode (US-5.5) |
| Edit mode body textarea | "Note" | Note card edit mode (US-5.5) |
| "Add Note" button | Button text is its own label | Compose box |
| "Save" button | Button text is its own label | Note card edit mode |
| "Cancel" button | Button text is its own label | Note card edit mode |
| "Edit" button | Should have `aria-label="Edit note: {title}"` | Note card (disambiguates multiple Edit buttons) |
| "Delete" button | Should have `aria-label="Delete note: {title}"` | Note card (disambiguates multiple Delete buttons) |

**Multiple Edit/Delete buttons:** Because the page may have many "Edit" buttons (one per card), each button should have a unique accessible label via `aria-label` that includes the note title (or "Untitled"). This allows screen reader users to distinguish them.

```html
<button aria-label="Edit note: My Meeting Note">Edit</button>
<button aria-label="Delete note: My Meeting Note">Delete</button>
```

---

## 2. Keyboard Navigation (US-3.4, US-5.5)

### Tab Order — Compose Box
1. Title `<input>` (optional, first Tab stop)
2. Body `<textarea>` (required, second Tab stop)
3. "Add Note" `<button>` (third Tab stop)

### Tab Order — Note Card (Read View)
Within each card, the Tab order should be logical:
1. "Edit" button
2. "Delete" button

### Tab Order — Note Card (Edit Mode)
1. Title `<input>` (pre-filled)
2. Body `<textarea>` (pre-filled)
3. "Save" `<button>`
4. "Cancel" `<button>`

### Keyboard Activation
- All buttons (`<button>` elements) are activatable via **Enter** and **Space** when focused (native HTML behaviour)
- Do not use `<div>` or `<span>` as buttons — use `<button>` elements (native keyboard behaviour)
- The `disabled` attribute on the "Add Note" and "Save" buttons must prevent keyboard activation, not just mouse click

### Focus Management
- When a note card enters edit mode, focus should ideally move to the Title input (or Body textarea) of that card
- When a note card exits edit mode (Save success or Cancel), focus returns to the "Edit" button on the card
- When a note is deleted, focus should move to the next card's "Delete" or "Edit" button, or to the compose box if no cards remain

---

## 3. ARIA Roles and Live Regions

### Error Messages
All inline error messages should use `role="alert"` or `aria-live="polite"` so screen readers announce them when they appear:

```html
<!-- Error zone — announced automatically when content changes -->
<p role="alert" aria-live="polite">
  Failed to save note. Please try again.
</p>
```

Use `role="alert"` for errors (immediate announcement) and `aria-live="polite"` for non-urgent status updates.

### Note List Loading State
```html
<div aria-live="polite" aria-label="Note list status">
  Loading notes…
</div>
```

### Empty State
```html
<p>No notes yet — add your first one above.</p>
```
This requires no special ARIA — it is plain text content and will be read by screen readers.

### Note Cards
```html
<article aria-label="Note: {title or 'Untitled'}">
  ...card contents...
</article>
```

Using `<article>` is semantically appropriate for self-contained note cards and provides screen reader context.

---

## 4. Semantic HTML

| Element | Use Case |
|---|---|
| `<h1>` | Page title "QuickNotes" — one per page |
| `<main>` | Main content area wrapping compose box + note list |
| `<form>` | Compose box (wraps inputs and submit button) |
| `<label>` | All form field labels |
| `<button>` | All interactive controls (never `<div onClick>`) |
| `<textarea>` | Multi-line text inputs (body fields) |
| `<input type="text">` | Single-line text inputs (title fields) |
| `<time dateTime="...">` | All relative timestamps (US-4.1) |
| `<article>` | Each note card |
| `<ul>` / `<li>` | Note card list container (optional; provides list semantics) |

**`<time>` element requirement (US-4.1):**
```html
<time dateTime="2026-07-04T12:00:00.000Z">2 minutes ago</time>
```
The `dateTime` attribute contains the machine-readable ISO 8601 UTC timestamp; the display text is the human-readable relative string.

---

## 5. Colour Contrast

- **Body text on background:** Minimum 4.5:1 contrast ratio (WCAG 2.1 AA for normal text)
- **"Untitled" fallback text:** If displayed in italic/muted colour, must maintain ≥ 4.5:1 contrast (or ≥ 3:1 if large text)
- **Disabled button:** May have reduced contrast (WCAG exception for disabled elements), but should still be visually distinguishable
- **Error messages:** Red/orange error text must maintain ≥ 4.5:1 against its background
- **Timestamp text:** If displayed in a lighter/smaller style, must still meet minimum contrast ratios

**Do not rely on colour alone** to communicate state (e.g., disabled state should not be conveyed by colour only — the native `disabled` attribute communicates this to assistive technology).

---

## 6. Server-Side Render Requirement (US-3.4)

The compose box must be present in the **initial server-rendered HTML** (Next.js SSR), not only after client-side JavaScript hydration. This ensures:
- Screen readers can access the compose box before JS loads
- Users with JS disabled or slow JS execution can see the form structure
- Search engine crawlers see the form (relevant if the app is ever made public)

The note list data is fetched client-side (React state after hydration), which is acceptable — the structure (list container, empty state placeholder) may be SSR'd.

---

## 7. Motion and Animation

- QuickNotes has minimal required animation. Card transitions (read → edit mode) should be **instant or very fast** (< 150ms) to avoid motion sickness
- Avoid `prefers-reduced-motion: reduce` violations — if any CSS transitions are added, wrap them in a media query respecting the user's motion preference:

```css
@media (prefers-reduced-motion: no-preference) {
  .card-transition { transition: all 0.15s ease; }
}
```

---

## Accessibility Checklist

- [ ] All form inputs have associated `<label>` elements (US-3.4, US-5.5)
- [ ] Multiple "Edit"/"Delete" buttons have unique `aria-label` values including note title
- [ ] Tab order is logical within compose box and note cards (US-3.4, US-5.5)
- [ ] All buttons are `<button>` elements (not `<div>`/`<span>`)
- [ ] "Add Note" and "Save" disabled state uses native HTML `disabled` attribute
- [ ] Error zones use `role="alert"` or `aria-live="polite"`
- [ ] Timestamps use `<time dateTime="...">` with ISO 8601 UTC value (US-4.1)
- [ ] Colour contrast meets 4.5:1 for all body text
- [ ] Compose box is in SSR initial HTML (US-3.4)
- [ ] Focus management is handled on edit mode enter/exit and card deletion
