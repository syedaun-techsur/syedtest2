# QuickNotes

## What This Is

QuickNotes is a minimal, single-user note-taking web app built with Next.js (App Router, TypeScript). Users can create, view, edit, and delete short text notes that persist to a PostgreSQL database so they survive server restarts. The app serves both the UI and its API routes from a single Next.js application on port 3000.

## Core Value

A user can capture a note in under 5 seconds and trust it will be there when they return — fast capture, reliable persistence.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] App scaffolds as Next.js (App Router, TypeScript), starts with `npm run dev` on port 3000
- [ ] GET /health returns 200 JSON `{ "status": "ok" }`
- [ ] Notes persist to PostgreSQL via `DATABASE_URL` environment variable (no Docker dependency)
- [ ] Database schema: `notes` table with `id` (uuid/serial PK), `title` (text, nullable), `body` (text, not null), `created_at` (timestamptz, default now()), `updated_at` (timestamptz, default now(), bumped on edit)
- [ ] GET /api/notes → list all notes, newest first
- [ ] POST /api/notes → create note `{ title?, body }`; 400 if body empty; returns created note
- [ ] GET /api/notes/:id → single note; 404 if missing
- [ ] PUT /api/notes/:id → update note `{ title?, body }`; 404 if missing
- [ ] DELETE /api/notes/:id → delete note; 404 if missing
- [ ] UI: Header "QuickNotes"
- [ ] UI: Compose box with optional title input, required body textarea, "Add Note" button (disabled when body is empty)
- [ ] UI: List of notes below compose box, newest first, each showing title (or "Untitled"), body, and relative timestamp
- [ ] UI: Inline edit and delete (with confirm) controls on each note
- [ ] UI: Empty state message "No notes yet — add your first one above."

### Out of Scope

- Authentication / multi-user accounts — single implicit user; auth adds complexity out of scope for v1
- Rich-text, tags, search, file attachments — future work
- Real-time sync across tabs — not needed for single-user experience
- Docker / docker-compose — runtime provisions database; app must connect natively

## Context

- **Runtime**: Database is provisioned by the environment and injected as `DATABASE_URL`. If `PIVOTA_DB_MODE` indicates a sidecar, connect directly and run migrations natively (no compose).
- **ORM/queries**: Use `pg` client or Prisma. If Prisma, run `prisma migrate deploy` or `prisma db push` against `DATABASE_URL` at startup — never via Docker.
- **Port**: Dev server must bind to `0.0.0.0:3000`.
- **Phase breakdown** (from PRD): 1. Scaffold & health, 2. DB + schema, 3. API routes, 4. UI, 5. Polish & UAT.

## Constraints

- **Tech Stack**: Next.js App Router, TypeScript — must match pinned major version for config file extension (no `next.config.ts` on Next 14)
- **Database**: PostgreSQL only, accessed via `DATABASE_URL` — no Docker assumption
- **Port**: 3000, bound to `0.0.0.0` for sandbox/preview compatibility
- **Migrations**: Run natively at startup, no compose
- **No auth**: Single implicit user — do not introduce authentication

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| PostgreSQL via DATABASE_URL (no Docker) | Runtime provisions DB; app must be portable | — Pending |
| Next.js App Router (not Pages Router) | Modern routing pattern, required by spec | — Pending |
| Single-user, no auth | Simplicity; auth is explicit non-goal | — Pending |
| Lightweight ORM (pg or Prisma) | Avoid heavy ORMs; prefer direct SQL or thin layer | — Pending |

---
*Last updated: 2026-07-04 after initialization*
