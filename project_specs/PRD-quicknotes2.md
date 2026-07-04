# Product Requirements Document
# QuickNotes

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Version:** 1.0  
**Date:** 2026-07-04  
**Status:** Active  

---

## 1. Executive Summary

QuickNotes is a minimal, single-user note-taking web application built with Next.js (App Router, TypeScript) and backed by PostgreSQL. It enables a user to capture, view, edit, and delete short text notes in seconds, with data persisting across server restarts. The application serves both the UI and its REST API from a single Next.js process on port 3000.

---

## 2. Problem Statement

Users frequently need to capture fleeting thoughts, tasks, or references quickly — but most note tools are overengineered, require accounts, or are too slow to open. There is a clear need for:

- **Fast capture**: A note should be saveable in under 5 seconds from app open.
- **Reliable persistence**: Notes must survive server restarts; in-memory storage is not acceptable.
- **Zero friction**: No login, no workspace setup, no formatting overhead — just write and save.
- **Accessible from anywhere**: A browser-based UI accessible on port 3000 without additional tooling.

QuickNotes solves these problems by providing a single-page, single-user web app where the compose box is always visible and notes are persisted immediately to PostgreSQL via environment-injected credentials.

---

## 3. Product Vision

**Vision Statement:** To be the fastest path from thought to persisted note — a tool so simple it disappears, leaving only the content.

**Strategic Goals:**
- Ship a fully functional note-taking MVP with zero authentication overhead
- Validate that the core capture → persist → retrieve loop is reliable enough for daily use
- Establish a clean Next.js App Router + PostgreSQL foundation that can be extended in future phases
- Demonstrate that a production-quality single-user app can be built and deployed in a single, portable Next.js process

---

## 4. Technical Architecture

| Layer | Technology | Notes |
|---|---|---|
| Frontend Framework | Next.js 14+ (App Router, TypeScript) | Single process serves UI and API |
| Language | TypeScript | Strict typing throughout |
| Styling | Tailwind CSS or CSS Modules | Minimal, functional styles |
| Database | PostgreSQL | Accessed via `DATABASE_URL` env var; no Docker dependency |
| Database Client | `pg` (node-postgres) or Prisma | Prefer direct SQL or thin layer; no heavy ORM |
| Migrations | Native at startup | `prisma migrate deploy` / `prisma db push` or raw SQL — never via Docker Compose |
| API | Next.js Route Handlers (`/app/api/`) | REST-style endpoints |
| Runtime Port | `0.0.0.0:3000` | Required for sandbox/preview compatibility |
| Dev Server | `npm run dev` | Starts on port 3000 |

---

## 5. Feature Requirements

### F0: Application Scaffold & Health Endpoint
**Description:** The application bootstraps as a Next.js App Router project with TypeScript. It starts cleanly with `npm run dev`, binds to `0.0.0.0:3000`, and exposes a health check endpoint so infrastructure can verify the app is running.

**Capabilities:**
- Next.js project initialised with App Router and TypeScript (`tsconfig.json`, `app/` directory)
- Dev server starts with `npm run dev` on port 3000, bound to `0.0.0.0`
- `GET /health` returns HTTP 200 with JSON body `{ "status": "ok" }`
- No Docker dependency — app runs with native Node.js

**Priority:** P0 (Critical — MVP prerequisite; nothing else works without scaffold)

---

### F1: Database Schema & Connectivity
**Description:** The application connects to PostgreSQL using the `DATABASE_URL` environment variable and ensures the `notes` table schema is in place at startup. Migrations run natively, not via Docker Compose.

**Capabilities:**
- Reads `DATABASE_URL` from environment; fails fast with a clear error if missing
- Creates/migrates `notes` table on startup with columns:
  - `id` — UUID or serial primary key
  - `title` — text, nullable
  - `body` — text, not null
  - `created_at` — timestamptz, default `now()`
  - `updated_at` — timestamptz, default `now()`, bumped on every edit
- Supports `PIVOTA_DB_MODE` sidecar mode: connects directly and runs migrations natively
- Migration strategy: Prisma (`prisma migrate deploy` / `prisma db push`) or raw SQL DDL — no Compose

**Priority:** P0 (Critical — all data features depend on this)

---

### F2: Notes REST API
**Description:** A full CRUD REST API for notes, implemented as Next.js Route Handlers under `/api/notes`. All endpoints return JSON and use standard HTTP status codes.

**Capabilities:**
- `GET /api/notes` — Returns all notes ordered by `created_at` descending (newest first)
- `POST /api/notes` — Creates a note from `{ title?, body }`; returns 400 if `body` is empty or missing; returns the created note with 201
- `GET /api/notes/:id` — Returns a single note; 404 if not found
- `PUT /api/notes/:id` — Updates a note from `{ title?, body }`; 404 if not found; bumps `updated_at`
- `DELETE /api/notes/:id` — Deletes a note; 404 if not found; returns 204 or success JSON
- All error responses return JSON `{ "error": "<message>" }` with the appropriate HTTP status code

**Priority:** P0 (Critical — UI depends on all CRUD operations)

---

### F3: Single-Page UI — Compose Box
**Description:** The primary interaction surface: a compose box always visible at the top of the page where users create new notes quickly.

**Capabilities:**
- Page header displays "QuickNotes" as the application title
- Optional title input field (text input, placeholder "Title (optional)")
- Required body textarea (placeholder e.g. "Write a note…")
- "Add Note" button — disabled when body field is empty or whitespace-only
- On submit: calls `POST /api/notes`, clears the form on success, and refreshes the note list
- Accessible: proper `<label>` associations, keyboard-navigable

**Priority:** P0 (Critical — core capture flow)

---

### F4: Single-Page UI — Note List
**Description:** Below the compose box, a live list of all persisted notes, newest first, giving the user immediate visibility into their captured notes.

**Capabilities:**
- Displays all notes fetched from `GET /api/notes` on page load
- Each note card shows:
  - Title (falls back to "Untitled" if null/empty)
  - Body text
  - Relative timestamp (e.g. "2 minutes ago", "yesterday")
- Notes ordered newest first
- Empty state: shows message "No notes yet — add your first one above." when no notes exist
- List updates immediately after create, edit, or delete without full page reload

**Priority:** P0 (Critical — primary read surface)

---

### F5: Single-Page UI — Inline Edit & Delete
**Description:** Each note card exposes inline controls allowing the user to edit or delete a note without navigating away.

**Capabilities:**
- Edit control: clicking "Edit" on a note card transitions it into an editable state (inline inputs for title and body pre-populated with current values)
- Save edit: calls `PUT /api/notes/:id`; updates card in place on success
- Cancel edit: reverts to read view without saving
- Delete control: clicking "Delete" shows a confirmation prompt ("Are you sure?"); on confirm, calls `DELETE /api/notes/:id` and removes the card from the list
- Keyboard accessible: edit/delete controls reachable via Tab key

**Priority:** P1 (High — required for full CRUD in UI; MVP-level but layered after list)

---

## 6. Non-Functional Requirements

| Category | Requirement | Target |
|---|---|---|
| Performance | Note creation round-trip (POST + UI update) | < 500 ms on local network |
| Performance | Page initial load (notes list rendered) | < 2 s on typical connection |
| Reliability | App restarts without data loss | 100% — PostgreSQL persistence required |
| Reliability | Startup fails fast if `DATABASE_URL` is missing | Immediate error, clear message |
| Portability | No Docker dependency in runtime | Must connect natively via `DATABASE_URL` |
| Portability | Binds to `0.0.0.0:3000` | Required for sandbox/preview environments |
| Correctness | API input validation | `POST /api/notes` returns 400 for empty body |
| Correctness | 404 responses for missing resources | All `:id` endpoints return 404 correctly |
| Maintainability | TypeScript strict mode | No `any` usage in production code |
| Maintainability | Single deployable unit | UI + API from one Next.js process |
| Security | No authentication scope | Single-user; no auth surface to secure in v1 |
| Browser Support | Modern evergreen browsers | Chrome, Firefox, Safari, Edge (latest) |

---

## 7. Success Metrics

- **Capture speed**: A note can be created (compose → submit → visible in list) in under 5 seconds from app open
- **Persistence reliability**: Notes are present after server restart in 100% of test cases
- **API correctness**: All 5 CRUD endpoints return correct HTTP status codes and JSON payloads in automated tests
- **Health endpoint**: `GET /health` returns `{ "status": "ok" }` with HTTP 200 within 100 ms
- **UI completeness**: All UI requirements (compose, list, edit, delete, empty state) are present and functional in manual UAT
- **Zero startup failures**: App starts successfully given a valid `DATABASE_URL` in 100% of runs; fails with a clear error in 100% of runs when `DATABASE_URL` is absent

---

## 8. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| `DATABASE_URL` misconfiguration in environment | Medium | High | Fail fast on startup with descriptive error; document env var requirement clearly |
| Migration conflicts on repeated startup | Low | Medium | Use idempotent DDL (`CREATE TABLE IF NOT EXISTS`) or Prisma's `migrate deploy` which is safe to rerun |
| Next.js config file extension mismatch (`.ts` vs `.js`) on Next 14 | Medium | Medium | Pin `next.config.js` (not `.ts`); check Next.js major version before scaffold |
| Port 3000 already in use | Low | Low | Document port requirement; `0.0.0.0` binding ensures sandbox compatibility |
| Feature creep (auth, search, rich text) | Low | Medium | Scope is locked in PROJECT.md; defer all v2 features explicitly |
| Body textarea UX — accidental empty submit | Low | Low | "Add Note" button disabled when body is empty; enforced server-side with 400 |

---

## 9. Out of Scope (v1)

The following are explicitly **not** part of this release:

- Authentication or multi-user accounts
- Rich-text editing, Markdown rendering
- Tags, categories, or search/filter
- File or image attachments
- Real-time sync across browser tabs
- Docker / docker-compose runtime
- Mobile-native app (web only)
- Note archiving, trash/restore

---

## 10. Feature Index

| ID | Feature | Priority | Phase | Depends On |
|---|---|---|---|---|
| F0 | Application Scaffold & Health Endpoint | P0 — Critical | 1: Scaffold & Health | — |
| F1 | Database Schema & Connectivity | P0 — Critical | 2: DB + Schema | F0 |
| F2 | Notes REST API | P0 — Critical | 3: API Routes | F1 |
| F3 | Single-Page UI — Compose Box | P0 — Critical | 4: UI | F2 |
| F4 | Single-Page UI — Note List | P0 — Critical | 4: UI | F2, F3 |
| F5 | Single-Page UI — Inline Edit & Delete | P1 — High | 4: UI + Polish | F4 |

---

## 11. Implementation Phases

| Phase | Name | Features | Exit Criteria |
|---|---|---|---|
| 1 | Scaffold & Health | F0 | `GET /health` returns 200 `{ "status": "ok" }` |
| 2 | DB + Schema | F1 | `notes` table created on startup; app connects via `DATABASE_URL` |
| 3 | API Routes | F2 | All 5 CRUD endpoints pass automated tests |
| 4 | UI | F3, F4, F5 | Manual UAT: create, view, edit, delete notes; empty state visible |
| 5 | Polish & UAT | All | Performance targets met; edge cases (404, 400, empty state) verified |

---

*Document generated: 2026-07-04*  
*Based on: `.planning/PROJECT.md`*  
*Next documents: FRD-quicknotes2.md, TechArch-quicknotes2.md, UserStories-quicknotes2.md*
