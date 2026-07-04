# User Journey Maps
# QuickNotes

| Field | Value |
|---|---|
| **Product Name** | QuickNotes |
| **Acronym** | quicknotes2 |
| **Version** | 1.0 |
| **Date** | 2026-07-04 |
| **Related Personas** | PERSONAS-quicknotes2.md (PER-01, PER-02) |
| **Related JTBD** | JTBD-quicknotes2.md (JTBD-01.1, JTBD-01.2, JTBD-01.3, JTBD-02.1, JTBD-02.2, JTBD-02.3) |
| **Related PRD** | PRD-quicknotes2.md |

---

## Journey Index

| JRN-ID | Persona | Scenario | Key JTBD | Stages |
|---|---|---|---|---|
| JRN-01.1 | PER-01 — Alex Rivera (Knowledge Worker) | Capturing a thought mid-meeting before it vanishes | JTBD-01.1 | 5 |
| JRN-01.2 | PER-01 — Alex Rivera (Knowledge Worker) | Returning to review and act on captured notes | JTBD-01.2 | 5 |
| JRN-01.3 | PER-01 — Alex Rivera (Knowledge Worker) | Correcting and retiring notes after they've been actioned | JTBD-01.3 | 5 |
| JRN-02.1 | PER-02 — Jordan Kim (Developer) | First-run deployment and operational health validation | JTBD-02.1, JTBD-02.2 | 6 |
| JRN-02.2 | PER-02 — Jordan Kim (Developer) | Exercising the full REST API contract for automated testing | JTBD-02.3 | 5 |

---

## PER-01: Alex Rivera — Knowledge Worker / Daily Note-Taker

---

### JRN-01.1: Capturing a Thought Mid-Meeting

**Persona:** PER-01 (Alex Rivera)

**Scenario:** Alex is in a video call when a follow-up task surfaces — something that must not be forgotten but cannot be typed into the meeting chat. Alex reaches for the QuickNotes browser tab (already open in the background), types the thought, submits, and returns to the meeting — all in under ten seconds. This is the most frequent and most critical flow in Alex's daily use. If this fails, Alex loses the thought and falls back to sticky notes or mental notes.

**Related Jobs:** JTBD-01.1

### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| **Switch** | Alt-tabs to the QuickNotes browser tab already open in background | Browser tab (existing session) | "Please don't make me wait — I need to type this NOW" | Anxious, time-pressured | Every second of load time risks losing the thought | App is already loaded; no reload needed if tab is kept open |
| **Orient** | Scans the top of the page and sees the compose box immediately | Compose box — F3 | "Good, it's right there — no clicking around" | Relieved, focused | If the compose box were below the fold, the thought would be lost before scrolling | Pin compose box to top of page; it must be above the fold on every viewport |
| **Type** | Clicks into the body textarea (or it is already focused) and types the note | Body textarea — F3 | "Let me just get this down fast — title doesn't matter right now" | Determined, fast-moving | Forced title fields or form wizards break flow | Body textarea auto-focuses on page load; title is optional and comes after |
| **Submit** | Clicks "Add Note" or presses Enter/Tab+Enter to submit | "Add Note" button — F3, POST /api/notes — F2 | "Did it save? Is the button doing something?" | Briefly anxious | If the button is slow or shows no feedback, Alex clicks again and creates a duplicate | Button shows immediate loading state; clears form on success; new note appears within 500 ms |
| **Confirm** | Sees the new note card appear at the top of the list below | Note list — F4 | "There it is — I can switch back to the meeting now" | Relieved, satisfied | If the list doesn't update instantly, Alex cannot tell if the note was saved | Optimistic UI or fast refresh to show the card immediately; no full page reload |

### Key Moments
- **Decision Point:** Submit stage — Alex must trust the app saved the note before switching back to the meeting; any ambiguity causes a second click or abandonment
- **Risk of Abandonment:** Switch stage — if the tab requires a page reload or the compose box is not immediately visible, Alex abandons and uses a sticky note instead
- **Delight Opportunity:** Confirm stage — the note card appearing instantly at the top of the list is the micro-moment of trust that builds daily habit; "it just worked"

### Success Outcome
Alex opens the tab, types a note, and sees it at the top of the list in under 5 seconds — matching JTBD-01.1 success measure and PRD Section 7 capture speed metric.

### Feature Touchpoints

| Stage | Features |
|---|---|
| Switch | F3 (Compose Box — page already loaded) |
| Orient | F3 (Compose Box visibility) |
| Type | F3 (Body textarea, optional title) |
| Submit | F3 (Add Note button), F2 (POST /api/notes) |
| Confirm | F4 (Note List — live update) |

---

### JRN-01.2: Returning to Review and Act on Captured Notes

**Persona:** PER-01 (Alex Rivera)

**Scenario:** An hour after a call, Alex opens QuickNotes to retrieve the three follow-up items captured earlier. Alex expects to see them immediately — newest first — without scrolling through stale noise or performing any search. The goal is to scan the list, identify the relevant card, and use its content to take the next real-world action (send an email, open a ticket, update a doc). If notes are missing or out of order, trust in the tool collapses.

**Related Jobs:** JTBD-01.2

### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| **Open** | Opens the QuickNotes browser tab or navigates to `http://localhost:3000` | Browser — page load | "Are my notes still there from this morning?" | Mildly anxious, slightly uncertain | If the server restarted overnight, are in-memory notes gone? | PostgreSQL persistence (F1) guarantees notes survive restarts; this anxiety fades after first successful return |
| **Load** | Page renders; note list populates from `GET /api/notes` | Note list — F4, GET /api/notes — F2 | "I need to see the most recent one at the top" | Scanning, focused | Alphabetical ordering or flat chronological (oldest first) breaks the mental model | Always render newest-first; API returns `ORDER BY created_at DESC` |
| **Scan** | Eyes move down the list of note cards; reads titles and timestamps | Note cards — F4 | "Which of these was the 'follow up with Sam' one? It was from 2 hours ago" | Focused, slightly pressured | Dense cards with no timestamp make it hard to identify which note is relevant | Each card shows a human-readable relative timestamp ("2 hours ago") and a title or "Untitled" fallback |
| **Identify** | Finds the target note card by timestamp or body preview | Note card — F4 | "There it is — 'follow up with Sam re: pricing'" | Relieved | If body is truncated aggressively, key content may not be visible at a glance | Show enough body text on the card to allow identification without clicking into a detail view |
| **Act** | Reads the note and takes the real-world next action; returns to app to delete or edit the note | Note card, then external action, then back to F4/F5 | "Done. I should delete this one so it doesn't clutter the list" | Satisfied, then transitioning to cleanup | No easy transition from "read note" to "delete or edit" — must find the controls | Edit and Delete controls visible directly on each card; no separate detail page needed |

### Key Moments
- **Decision Point:** Load stage — if the list is empty when notes should exist, Alex immediately loses trust and is unsure if the app is broken or notes were lost
- **Risk of Abandonment:** Scan stage — if the list is too long, unordered, or lacks timestamps, Alex cannot find the relevant note quickly and falls back to memory or other tools
- **Delight Opportunity:** Open stage — seeing all notes exactly as they were last session builds the "trust loop" that makes QuickNotes a daily habit rather than a temporary experiment

### Success Outcome
Alex identifies the target note within 10 seconds of opening the app with zero navigation steps — matching JTBD-01.2 success measure. Empty state message confirms the app is working when no notes exist.

### Feature Touchpoints

| Stage | Features |
|---|---|
| Open | F1 (DB persistence — notes survive restart) |
| Load | F4 (Note List), F2 (GET /api/notes) |
| Scan | F4 (Note cards — timestamp, title, body) |
| Identify | F4 (Note card body preview) |
| Act | F5 (Inline Edit & Delete controls) |

---

### JRN-01.3: Correcting and Retiring Notes After Action

**Persona:** PER-01 (Alex Rivera)

**Scenario:** Alex reviews the note list at the end of the week. Several notes have typos from hasty capture ("followu w/ Sam"), one note was captured with incomplete context ("check contract — which one?"), and two notes have been fully actioned. Alex wants to fix the typo, expand the incomplete note with the missing detail, and delete the two completed ones — all without leaving the main page or navigating to a separate edit screen. The faster this cleanup takes, the more likely it becomes a regular habit.

**Related Jobs:** JTBD-01.3

### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| **Select** | Scans note list and identifies a note that needs editing; clicks "Edit" on its card | Note card "Edit" control — F5 | "I need to fix that typo before I forget what I meant" | Mildly frustrated by the error, motivated to fix it | If "Edit" is hard to find or requires hovering, the effort threshold rises | "Edit" button visible on each card, reachable by Tab key; no hover-only controls |
| **Edit** | Card transitions to inline edit mode; title and body inputs are pre-populated | Inline edit inputs — F5 | "Good — I don't have to retype the whole thing, just change the one word" | Focused, efficient | Blank edit fields requiring full re-entry would be a deal-breaker for this persona | Pre-populate both title and body with current values; cursor positioned at end of body |
| **Save** | Corrects the text and clicks "Save"; card updates in place | "Save" button — F5, PUT /api/notes/:id — F2 | "Did it save? Let me check the card looks right" | Briefly uncertain, then confirmed | If the card doesn't update immediately, Alex may click Save again causing a duplicate request | Card updates in place within 500 ms of save; loading state on button prevents double-submission |
| **Cancel** | Clicks "Cancel" on a note where Alex changed their mind mid-edit | "Cancel" button — F5 | "Actually, I'll leave that one for now" | Relaxed — exit is safe | If Cancel requires confirmation or is hard to find, edit mode feels like a trap | Cancel button is clearly visible next to Save; clicking it immediately reverts to read view with no changes |
| **Delete** | Finds a completed note and clicks "Delete"; reads confirmation; clicks confirm | "Delete" button, confirmation prompt — F5, DELETE /api/notes/:id — F2 | "I'm sure I want to delete this one — it's done" | Purposeful, slightly cautious | No confirmation = risk of accidental permanent loss; too many confirmation steps = friction | Single confirmation prompt ("Are you sure?"); confirm removes the card immediately; cancel keeps it |

### Key Moments
- **Decision Point:** Delete stage — the confirmation prompt is the only safety net; it must be present but not disruptive (not a modal that blocks the whole page)
- **Risk of Abandonment:** Edit stage — if edit mode is slow to open or fields are blank instead of pre-populated, Alex closes the tab and lives with the imperfect note
- **Delight Opportunity:** Save stage — seeing the corrected content update in the card in place (no page reload, no flash) makes the product feel polished and trustworthy

### Success Outcome
Alex edits a note body and sees the corrected content in the card within 10 seconds of clicking "Edit" — no page navigation required. Matching JTBD-01.3 success measure.

### Feature Touchpoints

| Stage | Features |
|---|---|
| Select | F5 (Edit control on card) |
| Edit | F5 (Inline edit inputs, pre-populated) |
| Save | F5 (Save button), F2 (PUT /api/notes/:id) |
| Cancel | F5 (Cancel button) |
| Delete | F5 (Delete button + confirmation), F2 (DELETE /api/notes/:id) |

---

## PER-02: Jordan Kim — Developer / Technical Evaluator

---

### JRN-02.1: First-Run Deployment and Operational Health Validation

**Persona:** PER-02 (Jordan Kim)

**Scenario:** Jordan has just cloned the QuickNotes repository into a fresh environment — a local machine, a CI sandbox, or a shared dev server. The PostgreSQL database is already provisioned by the environment and exposed as `DATABASE_URL`. Jordan's goal is to get from `git clone` to a verified, healthy running app as efficiently as possible, confirming at each step that there are no hidden dependencies, silent failures, or manual workarounds. If the app doesn't start cleanly in one command, Jordan considers it unshippable.

**Related Jobs:** JTBD-02.1, JTBD-02.2

### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| **Clone & Install** | Runs `git clone`, then `npm install` | Terminal, package.json | "Let me check if there are any unexpected peer dependency warnings or missing engines" | Methodical, watchful | Excessive `npm warn` output or engine mismatches slow down first-run confidence | Clean `npm install` with minimal warnings; locked `package-lock.json` ensures reproducibility |
| **Configure** | Sets `DATABASE_URL` in the environment (`.env.local` or shell export) | Terminal, .env.local | "Is this the only env var I need? Are there others I'll discover at runtime?" | Slightly cautious | Discovering additional required env vars at runtime (not at startup) is a trust-killer | Fail fast at startup for ALL required env vars; document them clearly in README |
| **Start** | Runs `npm run dev`; watches terminal output | Terminal — F0, F1 | "Does it bind to 0.0.0.0? Did migrations run? Any schema errors?" | Attentive, scanning logs | Silent startup that completes without confirming DB connection gives Jordan no confidence | Log a clear startup message confirming DB connected and schema migrated (or already up to date) |
| **Health Check** | Runs `curl http://localhost:3000/health` | GET /health — F0 | "I want JSON, not an HTML redirect. I want 200, not a 302" | Precise, exacting | `/health` returning an HTML page, a redirect, or non-200 status breaks any automated health probe | Return `{ "status": "ok" }` with HTTP 200 and `Content-Type: application/json`; no redirects |
| **Schema Verify** | Opens `psql` or runs a query to inspect the `notes` table schema | PostgreSQL — F1 | "Are all the required columns there? Are types correct? Created_at has a default?" | Methodical | Manual psql inspection is slow; if schema is wrong, identifying which migration failed is painful | Auto-migration at startup means schema is always present; idempotent DDL means re-running is safe |
| **Restart Test** | Stops and restarts `npm run dev`; checks logs and health endpoint again | Terminal — F0, F1 | "Will it fail on second run with a duplicate table error?" | Alert, testing edge case | Non-idempotent migrations fail on second startup with `table already exists` errors | `CREATE TABLE IF NOT EXISTS` or Prisma `migrate deploy` is idempotent; second startup is clean |

### Key Moments
- **Decision Point:** Start stage — the first `npm run dev` output is Jordan's primary signal of code quality; noisy or confusing logs reduce confidence in the codebase
- **Risk of Abandonment:** Configure stage — if Jordan discovers a second or third required env var only at runtime (not at startup), the fail-fast contract is broken; Jordan may abandon the stack
- **Delight Opportunity:** Health Check stage — receiving a clean `{ "status": "ok" }` from `curl` within 1 second of starting confirms the entire scaffold works; this is the moment Jordan decides to go deeper

### Success Outcome
Jordan verifies app health with a single `curl` call within 30 seconds of `npm run dev` completing, receiving HTTP 200 `{ "status": "ok" }` — matching JTBD-02.1 and JTBD-02.2 success measures.

### Feature Touchpoints

| Stage | Features |
|---|---|
| Clone & Install | F0 (Scaffold — project structure, package.json) |
| Configure | F1 (DATABASE_URL requirement) |
| Start | F0 (npm run dev, 0.0.0.0:3000), F1 (DB connect + migrations) |
| Health Check | F0 (GET /health) |
| Schema Verify | F1 (notes table schema) |
| Restart Test | F1 (idempotent migrations) |

---

### JRN-02.2: Exercising the Full REST API Contract

**Persona:** PER-02 (Jordan Kim)

**Scenario:** With the app running and health confirmed, Jordan moves to API contract validation. This is either a manual exercise (using `curl` from the terminal) or the execution of an automated test suite. Jordan works through each of the five CRUD endpoints systematically — create, list, get-by-id, update, delete — then exercises the error cases: empty body on POST (expect 400), unknown ID on GET/PUT/DELETE (expect 404). Every endpoint must return the documented status code, a JSON body in the expected shape, and a consistent error envelope. Any deviation — a 200 where 201 is expected, a plain-text error where JSON is required — will be flagged.

**Related Jobs:** JTBD-02.3

### Journey Stages

| Stage | Action | Touchpoint | Thinking | Feeling | Pain Point | Opportunity |
|---|---|---|---|---|---|---|
| **Create** | Sends `POST /api/notes` with `{ "body": "Test note" }`; inspects response | POST /api/notes — F2 | "I need 201, not 200. Does the response include `id` and `created_at`?" | Precise, evaluating | A 200 instead of 201 is a contract violation that breaks test assertions downstream | Return HTTP 201 with the full created note JSON including `id`, `title`, `body`, `created_at`, `updated_at` |
| **List** | Sends `GET /api/notes`; inspects response array and ordering | GET /api/notes — F2 | "Is it an array? Is the note I just created at index 0? Is it ordered newest-first?" | Methodical | API returning an object instead of an array, or notes in wrong order, breaks client code | Return a JSON array ordered by `created_at DESC`; newest note is always at index 0 |
| **Get by ID** | Sends `GET /api/notes/:id` with the ID from the Create step | GET /api/notes/:id — F2 | "Does the single-note response match what was in the list?" | Cross-referencing | Inconsistent field names or formats between list and get-by-id responses cause integration bugs | Single-note response is identical in shape to the notes in the list array |
| **Update** | Sends `PUT /api/notes/:id` with `{ "body": "Updated note" }`; inspects response | PUT /api/notes/:id — F2 | "Did `updated_at` actually change? Is the response the full updated note?" | Precise, verifying | `updated_at` not bumping on PUT is a subtle contract violation that breaks audit trails | Return the full updated note JSON; `updated_at` is always bumped; HTTP 200 |
| **Error Cases** | Sends `POST` with empty body (expect 400), `GET/PUT/DELETE` with unknown ID (expect 404) | All /api/notes endpoints — F2 | "Are error responses JSON or plain text? Is the envelope `{ 'error': '...' }` consistently?" | Exacting, test-minded | Any endpoint returning an HTML error page or plain-text string breaks automated test assertions | Every error returns `{ "error": "<message>" }` with correct status; no HTML, no bare strings |

### Key Moments
- **Decision Point:** Create stage — the HTTP status code (201 vs 200) is the first signal of whether the API was built to spec; if it returns 200, Jordan immediately questions all other endpoints
- **Risk of Abandonment:** Error Cases stage — if any error endpoint returns HTML (Next.js default error page) instead of JSON, Jordan considers the API unfit for automated testing and stops evaluation
- **Delight Opportunity:** Update stage — seeing `updated_at` correctly bumped in the response confirms the developer who built this understood the spec precisely; Jordan's confidence in the codebase increases

### Success Outcome
Jordan runs an automated test suite covering all 5 CRUD endpoints plus 400 and 404 edge cases and achieves 100% pass rate — matching JTBD-02.3 success measure, with no manual intervention beyond setting `DATABASE_URL`.

### Feature Touchpoints

| Stage | Features |
|---|---|
| Create | F2 (POST /api/notes), F1 (DB write) |
| List | F2 (GET /api/notes), F1 (DB read) |
| Get by ID | F2 (GET /api/notes/:id) |
| Update | F2 (PUT /api/notes/:id), F1 (updated_at) |
| Error Cases | F2 (400/404 error envelopes on all endpoints) |

---

## Cross-Journey Patterns

### CP-01: The "Just Works" Trust Loop
**Appears in:** JRN-01.1 (Confirm stage), JRN-01.2 (Open stage), JRN-02.1 (Health Check stage)

The single most important pattern across all journeys is the moment a user — whether Alex or Jordan — receives a signal that the system worked exactly as expected. For Alex, this is seeing the new note card appear immediately after submit. For Jordan, this is `{ "status": "ok" }` from `curl`. These micro-confirmations build the trust that converts a one-time trial into daily use or confident deployment. Any delay, ambiguity, or silence at these moments is disproportionately damaging.

**Design implication:** Every state change (note created, note updated, note deleted, health check) must produce an immediate, unambiguous visual or API signal of success.

---

### CP-02: Fail-Fast Is a Feature, Not an Edge Case
**Appears in:** JRN-02.1 (Configure and Start stages), JRN-02.2 (Error Cases stage), indirectly JRN-01.1 (Submit stage)

Both personas benefit from fast, clear failure signals — Jordan explicitly (missing `DATABASE_URL`, 400/404 error envelopes), Alex implicitly (disabled "Add Note" button prevents a bad submit). The pattern: failing loudly and immediately is always better than a silent hang or an ambiguous state. Every component in the system — startup, API validation, UI controls — should refuse to proceed silently when preconditions are not met.

**Design implication:** Implement fail-fast at every layer: environment validation at startup, input validation on POST, 404 on unknown IDs, disabled button on empty textarea, confirmation prompt before delete.

---

### CP-03: No Navigation Required
**Appears in:** JRN-01.1 (all stages), JRN-01.2 (Scan/Identify stages), JRN-01.3 (Select/Edit/Save stages)

The single-page design is not a technical choice — it is a persona constraint. Alex cannot afford to navigate to a "new note" page, a "note detail" page, or an "edit page." Every action must be completable on the single root page (`/`). This pattern governs F3, F4, and F5 entirely: compose box always visible at top, note list always visible below, edit mode inline on the card, delete confirmation inline. Any flow that requires a route change breaks Alex's use case.

**Design implication:** Route the entire UI on `/`. No `/notes/new`, no `/notes/:id` detail page, no `/notes/:id/edit`. All interactions are in-place on the root page.

---

### CP-04: Timestamps Are a Navigation Aid, Not a Decoration
**Appears in:** JRN-01.2 (Scan and Identify stages), JRN-01.3 (Select stage)

Alex uses relative timestamps ("2 hours ago", "yesterday") as the primary way to identify which note is which when titles are absent or generic. Timestamps are functional navigation, not decorative metadata. This elevates the timestamp display requirement from a "nice to have" to a core retrieval mechanism.

**Design implication:** Every note card must display a human-readable relative timestamp. The timestamp must update without a full page reload when notes are created or edited. "Untitled" fallback is also critical — cards with no title must still be identifiable by timestamp + body preview.

---

## Journey-to-JTBD Traceability

| JRN-ID | Stage | JTBD-ID | Expected Outcome |
|---|---|---|---|
| JRN-01.1 | Switch | JTBD-01.1 | Tab is already loaded; no reload delay |
| JRN-01.1 | Orient | JTBD-01.1 | Compose box visible above the fold immediately |
| JRN-01.1 | Type | JTBD-01.1 | Body textarea is primary; title is optional |
| JRN-01.1 | Submit | JTBD-01.1 | POST /api/notes called; form clears on success |
| JRN-01.1 | Confirm | JTBD-01.1 | New note card appears at top of list within 500 ms |
| JRN-01.2 | Open | JTBD-01.2 | Notes survive server restart (PostgreSQL persistence) |
| JRN-01.2 | Load | JTBD-01.2 | GET /api/notes returns all notes newest-first on page load |
| JRN-01.2 | Scan | JTBD-01.2 | Each card shows title, body preview, relative timestamp |
| JRN-01.2 | Identify | JTBD-01.2 | Most recent note identifiable within 10 seconds |
| JRN-01.2 | Act | JTBD-01.2 | Edit and Delete controls reachable from card without navigation |
| JRN-01.3 | Select | JTBD-01.3 | Edit control visible on card; reachable via Tab |
| JRN-01.3 | Edit | JTBD-01.3 | Inline edit mode pre-populates title and body |
| JRN-01.3 | Save | JTBD-01.3 | PUT /api/notes/:id; card updates in place within 500 ms |
| JRN-01.3 | Cancel | JTBD-01.3 | Reverts to read view; no changes persisted |
| JRN-01.3 | Delete | JTBD-01.3 | Confirmation prompt; DELETE /api/notes/:id; card removed from list |
| JRN-02.1 | Clone & Install | JTBD-02.1 | Clean npm install; no unexpected peer dependency errors |
| JRN-02.1 | Configure | JTBD-02.2 | DATABASE_URL is the only required env var; no runtime discovery |
| JRN-02.1 | Start | JTBD-02.1, JTBD-02.2 | npm run dev binds to 0.0.0.0:3000; migrations run automatically |
| JRN-02.1 | Health Check | JTBD-02.1 | GET /health returns HTTP 200 `{ "status": "ok" }` within 100 ms |
| JRN-02.1 | Schema Verify | JTBD-02.2 | notes table with all required columns present after startup |
| JRN-02.1 | Restart Test | JTBD-02.2 | Second startup is clean; no duplicate-table schema errors |
| JRN-02.2 | Create | JTBD-02.3 | POST /api/notes returns HTTP 201 with full note JSON |
| JRN-02.2 | List | JTBD-02.3 | GET /api/notes returns JSON array ordered newest-first |
| JRN-02.2 | Get by ID | JTBD-02.3 | GET /api/notes/:id returns HTTP 200 with note JSON; 404 for unknown |
| JRN-02.2 | Update | JTBD-02.3 | PUT /api/notes/:id returns HTTP 200; updated_at is bumped |
| JRN-02.2 | Error Cases | JTBD-02.3 | All error responses use `{ "error": "<message>" }` JSON envelope |

---

*Document generated: 2026-07-04*
*Derived from: PERSONAS-quicknotes2.md, JTBD-quicknotes2.md, PRD-quicknotes2.md, .planning/PROJECT.md*
*Next documents: STORY-MAP-quicknotes2.md, FRD-quicknotes2.md*
