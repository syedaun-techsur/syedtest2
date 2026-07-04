# Persona Document
# QuickNotes

| Field | Value |
|---|---|
| **Product Name** | QuickNotes |
| **Acronym** | quicknotes2 |
| **Version** | 1.0 |
| **Date** | 2026-07-04 |
| **Related PRD** | PRD-quicknotes2.md |
| **Derived From** | PRD Section 2 (Problem Statement), Section 3 (Product Vision), Section 5 (Features), Section 7 (Success Metrics) |

> **Note on Target Users:** The PRD does not include an explicit Section 2.2 Target Users table. Personas are derived from the Problem Statement's user needs ("Users frequently need to capture fleeting thoughts, tasks, or references quickly"), the single-user product design, and the deployment context (DATABASE_URL, health endpoint, technical setup). Two distinct personas are identified: the daily knowledge worker who uses the app, and the developer who deploys and operates it.

---

## Persona Summary

| ID | Representative Name | Role | Primary Goal |
|---|---|---|---|
| PER-01 | Alex Rivera | Knowledge Worker / Daily Note-Taker | Capture fleeting thoughts and tasks in under 5 seconds, with confidence they'll persist |
| PER-02 | Jordan Kim | Developer / Technical Evaluator | Deploy and validate the app quickly; confirm the stack is reliable before recommending or extending it |

---

## PER-01: Alex Rivera

**Role & Context:**
Alex is a knowledge worker — a product manager, consultant, researcher, or analyst — who operates in a fast-moving environment with a constant stream of ideas, action items, and references. Alex keeps a browser tab open at all times for QuickNotes and reaches for it during meetings, while reading, or mid-task when a thought surfaces that must not be lost. Alex has been burned by heavyweight tools: Notion takes 4 seconds to load a new page, Apple Notes requires switching apps, and sticky notes get lost. What Alex wants is a box that's already open, ready to receive text, with no decisions to make before typing.

Alex is not a developer. Alex uses QuickNotes purely through the browser UI and has no awareness of the API, the database, or the server. Alex's primary relationship with the product is through the compose box and the note list. The app is likely running locally on Alex's machine (or on a shared server), accessed via `http://localhost:3000`.

**Goals:**
- Get a thought or task into a note in under 5 seconds from opening the browser tab (F3 — Compose Box)
- See all captured notes in reverse-chronological order without any navigation (F4 — Note List)
- Correct or expand a note after the fact without losing the original entry (F5 — Inline Edit)
- Remove notes that are no longer relevant without confirmation overhead (F5 — Delete with confirm)
- Trust that notes are still there after closing the laptop or restarting the server (F1 — DB persistence)

**Pain Points** *(derived from PRD Section 2 Problem Statement)*:
- Most note tools are overengineered: require account creation, workspace setup, or initial configuration before a single note can be written
- Heavy apps are too slow to open — by the time the tool loads, the thought is gone
- In-memory or local-only notes disappear unexpectedly after a server restart or browser refresh
- Formatting requirements (Markdown, rich text) add friction to a task that should be instantaneous
- No single always-visible compose surface — having to navigate to a "new note" page breaks flow

**Technical Expertise:** Low-to-intermediate — comfortable using web apps in a browser; does not interact with terminals, environment variables, or APIs; expects the app to "just work" when a URL is opened.

**Top Tasks:**
1. Write a new note using the compose box and submit it (daily, multiple times — critical path)
2. Scan the note list to retrieve a recently captured thought (daily, high frequency)
3. Edit a note inline to correct a typo or add context (several times per week, medium)
4. Delete a note that has been actioned or is no longer relevant (weekly, low urgency)
5. Verify that old notes are still present after returning to the app (as-needed, trust-building)

**Success Criteria:**
- A new note is composed and visible in the list in under 5 seconds from page open (PRD Section 7: Capture speed)
- Notes captured yesterday are present when Alex opens the app today — zero data loss across server restarts (PRD Section 7: Persistence reliability)
- The compose box is immediately visible and focusable without any navigation step (F3 design requirement)
- The note list shows the most recent note at the top, with a readable relative timestamp (F4 requirement)
- No accidental blank note submissions — the "Add Note" button is non-activatable when the body is empty (F3 — disabled state)

---

## PER-02: Jordan Kim

**Role & Context:**
Jordan is a software developer — a full-stack engineer, DevOps practitioner, or technical lead — who is responsible for getting QuickNotes running in a target environment. This may mean deploying it locally for personal use, setting it up on a shared development server, evaluating it as a reference implementation of Next.js App Router + PostgreSQL, or extending it as a foundation for a more capable product in a future phase.

Jordan interacts with QuickNotes at multiple layers: the file system (project structure, `tsconfig.json`, `next.config.js`), the environment (`DATABASE_URL`, `PIVOTA_DB_MODE`), the process (`npm run dev`, port 3000), and the API (`GET /health`, `GET /api/notes`, `POST /api/notes`, etc.). Jordan reads source code, runs migrations, checks the health endpoint, and may write automated tests against the REST API. Jordan cares deeply about whether the app starts cleanly, fails with a useful error message, and has no hidden runtime dependencies (no Docker assumption, no `.env` hard-coded defaults).

Jordan is not the primary end-user of the note-taking UI, but evaluates whether it functions correctly. Jordan is also the person who will extend the app in future phases if the foundation is sound.

**Goals:**
- Start the application with a single command (`npm run dev`) and have it be fully functional within seconds (F0 — Scaffold)
- Confirm the app is healthy via a machine-readable health endpoint before routing traffic to it (F0 — `GET /health`)
- Verify that database connectivity and schema migration run automatically at startup without manual steps (F1 — DB & Migrations)
- Exercise all 5 CRUD REST API endpoints and confirm correct status codes and JSON payloads (F2 — Notes REST API)
- Trust that a misconfigured `DATABASE_URL` produces a clear, actionable error — not a silent hang or cryptic stack trace (F1 — fail-fast requirement)
- Evaluate the codebase as a clean Next.js App Router + PostgreSQL reference for future extension (F0, F1, F2 combined)

**Pain Points** *(derived from PRD Section 2 and Section 8 Risks)*:
- Apps that silently fail on missing environment variables waste debugging time — Jordan expects immediate, descriptive startup errors
- Hidden Docker dependencies in "simple" apps break portability and complicate deployment in environments that provision databases externally
- Config file extension mismatches (`next.config.ts` vs `.js` on Next 14) cause silent build failures that are hard to diagnose
- Inconsistent API behavior (wrong status codes, missing error JSON) makes automated testing unreliable
- Migration scripts that are not idempotent cause schema conflicts on repeated startup, breaking CI/CD pipelines

**Technical Expertise:** High — proficient with Node.js, TypeScript, Next.js, PostgreSQL, REST APIs, terminal tooling, and environment variable management. Reads source code directly. May write curl commands or automated tests against the API.

**Top Tasks:**
1. Clone, install dependencies, set `DATABASE_URL`, and run `npm run dev` — confirm clean startup (first-run, critical)
2. Call `GET /health` to confirm the app is live and the endpoint returns expected JSON (first-run and monitoring, high)
3. Verify DB migration ran correctly by inspecting the `notes` table schema after startup (first-run, high)
4. Exercise all REST API endpoints (`GET`, `POST`, `PUT`, `DELETE /api/notes`) and confirm status codes match the spec (validation, high)
5. Confirm the app fails fast with a useful error when `DATABASE_URL` is unset or malformed (edge-case validation, medium)

**Success Criteria:**
- `npm run dev` starts cleanly and the app is reachable at `http://localhost:3000` with zero manual intervention beyond setting `DATABASE_URL` (PRD Section 7: Zero startup failures)
- `GET /health` returns `{ "status": "ok" }` with HTTP 200 within 100 ms (PRD Section 7: Health endpoint)
- All 5 CRUD endpoints return correct HTTP status codes and JSON in automated tests (PRD Section 7: API correctness)
- Missing `DATABASE_URL` produces an immediate startup error with a clear message — no silent hang (PRD Section 6 NFR: Reliability)
- The app binds to `0.0.0.0:3000` and is reachable in sandbox/preview environments without additional network configuration (PRD Section 6 NFR: Portability)
- Migrations are idempotent — restarting the app multiple times does not produce schema errors (PRD Section 8 Risk: Migration conflicts)

---

## Persona Relationships

| Interaction | PER-01 (Alex — Note-Taker) | PER-02 (Jordan — Developer) |
|---|---|---|
| **Who sets up the app** | Does not set up — expects it to already be running | Sets up: installs, configures `DATABASE_URL`, starts the process |
| **Primary interface** | Browser UI (compose box, note list, edit/delete controls) | Terminal + browser (startup logs, `/health`, `/api/notes` endpoints, source code) |
| **Who benefits from persistence** | Alex — trusts notes survive restarts | Jordan — validates persistence works correctly |
| **Who triggers API calls** | Indirectly, via UI interactions | Directly, via curl, Postman, or automated tests |
| **Who cares about error messages** | Low — wants UI to work silently | High — startup errors and API error JSON are critical signals |
| **Dependency direction** | Alex depends on Jordan having deployed a working instance | Jordan's work enables Alex's daily use |

> In a solo-developer scenario, Jordan and Alex may be the same person wearing different hats at different times: the developer who set up the app on Friday becomes the note-taker on Monday.

---

## Feature-Persona Matrix

| Feature | Description | PER-01 (Alex — Note-Taker) | PER-02 (Jordan — Developer) |
|---|---|---|---|
| **F0** | Application Scaffold & Health Endpoint | None — transparent to UI user | **Primary** — must start cleanly; `/health` is a key validation step |
| **F1** | Database Schema & Connectivity | **Secondary** — benefits from persistence; unaware of implementation | **Primary** — validates `DATABASE_URL` config, migration idempotency, fail-fast error |
| **F2** | Notes REST API | **Secondary** — uses API indirectly via UI | **Primary** — exercises all 5 CRUD endpoints, validates status codes and error JSON |
| **F3** | Single-Page UI — Compose Box | **Primary** — core capture surface used multiple times daily | **Secondary** — validates UI renders correctly and POST is wired to API |
| **F4** | Single-Page UI — Note List | **Primary** — primary read surface for retrieved notes | **Secondary** — validates list renders and updates on data change |
| **F5** | Single-Page UI — Inline Edit & Delete | **Primary** — used to correct and clean up notes | **Secondary** — validates PUT and DELETE are wired correctly to UI controls |

**Matrix Key:** Primary = feature directly serves this persona's core goals | Secondary = persona benefits from or validates the feature | None = feature is transparent or irrelevant to this persona

---

*Document generated: 2026-07-04*
*Derived from: PRD-quicknotes2.md, .planning/PROJECT.md*
*Next documents: FRD-quicknotes2.md, UserStories-quicknotes2.md*
