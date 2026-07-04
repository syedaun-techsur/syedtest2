# Functional Requirements Document
# QuickNotes

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Version:** 1.0  
**Date:** 2026-07-04  
**Status:** Draft  
**Based On:** PRD-quicknotes2.md v1.0  

---

## Scope

This FRD specifies the functional behaviour of every feature in the QuickNotes MVP. It is intended as the single source of truth for developers: each section defines inputs, outputs, validation rules, process steps, error states, and the API/schema surface required to implement the feature without ambiguity. Out-of-scope items from the PRD (auth, rich text, search, Docker, multi-user) are not detailed here; any implementation deviation requires a PRD change first.

---

## How to Read This Document

- **Feature IDs** follow the PRD: `F0`–`F5`. Chunk files use zero-padded prefixes (`F00`–`F05`) for correct sort order.
- **Cross-references** use the form `see F02 §Process step 3` or `see Y1-api.md §Notes`.
- **API tables** in feature chunks are summaries; full request/response schemas are in `Y1-api.md`.
- **DDL** in feature chunks is a reference; canonical DDL is in `Y0-schema.md`.
- **Error tables** in feature chunks list feature-specific errors; the cross-feature catalog is in `Y2-errors.md`.
- All API endpoints return `Content-Type: application/json`.
- All error bodies have the shape `{ "error": "<human-readable message>" }`.
- TypeScript strict mode applies; no `any` in production code.

---

## Cross-Cutting Terminology

| Term | Definition |
|------|-----------|
| **Note** | A persisted record with an optional title and required body, stored in the `notes` table |
| **Body** | The required free-text content of a note (non-null, non-empty after trimming) |
| **Title** | An optional short label for a note; `null` or empty string is stored as `null`; displayed as "Untitled" in the UI |
| **DATABASE_URL** | PostgreSQL connection string injected via environment variable; required at startup |
| **PIVOTA_DB_MODE** | Optional env var signaling sidecar DB mode; when set, app connects directly and runs migrations natively |
| **Route Handler** | Next.js App Router API handler under `app/api/` |
| **Relative timestamp** | Human-readable time-since string (e.g. "2 minutes ago", "yesterday") rendered client-side |
| **Empty state** | UI condition when no notes exist; displays the message "No notes yet — add your first one above." |
| **Idempotent migration** | DDL designed to be safe to re-run on every startup (e.g. `CREATE TABLE IF NOT EXISTS`) |

---

## Master Table of Contents

| Chunk | Feature / Section |
|-------|------------------|
| [F00-scaffold-health.md](F00-scaffold-health.md) | F0: Application Scaffold & Health Endpoint |
| [F01-database-schema-connectivity.md](F01-database-schema-connectivity.md) | F1: Database Schema & Connectivity |
| [F02-notes-rest-api.md](F02-notes-rest-api.md) | F2: Notes REST API |
| [F03-compose-box-ui.md](F03-compose-box-ui.md) | F3: Single-Page UI — Compose Box |
| [F04-note-list-ui.md](F04-note-list-ui.md) | F4: Single-Page UI — Note List |
| [F05-inline-edit-delete-ui.md](F05-inline-edit-delete-ui.md) | F5: Single-Page UI — Inline Edit & Delete |
| [Y0-schema.md](Y0-schema.md) | Database Schema (full DDL) |
| [Y1-api.md](Y1-api.md) | REST API Endpoint Catalog |
| [Y2-errors.md](Y2-errors.md) | Cross-Feature Error Catalog |
| [Y3-integrations.md](Y3-integrations.md) | External Integration Points |

---
