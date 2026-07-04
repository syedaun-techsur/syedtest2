---

## F00: Application Scaffold & Health Endpoint

**Description:** This feature establishes the foundational Next.js project structure that all other features build upon. It initialises the App Router with TypeScript, starts the dev server on `0.0.0.0:3000`, and exposes a `/health` endpoint so infrastructure and operators can verify the application is live without triggering any database logic.

**Priority:** P0 — Critical. Nothing else can function without the scaffold.

**Depends On:** None.

---

### Terminology

- **App Router:** The Next.js 14+ directory-based routing system using the `app/` directory (as opposed to the legacy `pages/` directory).
- **Route Handler:** A file named `route.ts` inside `app/api/<path>/` that handles HTTP requests — the App Router equivalent of API routes.
- **`0.0.0.0` binding:** Listening on all network interfaces, required for sandbox and preview environments that proxy to the container's IP rather than `localhost`.
- **Health endpoint:** A lightweight HTTP endpoint that returns a machine-readable status with no side effects and no database dependency.

---

### Sub-features

- Next.js App Router project scaffolded with TypeScript (`tsconfig.json`, strict mode enabled)
- Dev server starts via `npm run dev` on `0.0.0.0:3000`
- `GET /health` Route Handler returning `{ "status": "ok" }` with HTTP 200
- No Docker or external runtime dependency for startup

---

### Process

1. Developer runs `npm run dev` (or `npm run build && npm start` in production).
2. Next.js reads `next.config.js` (`.js` extension required for Next.js 14 compatibility — not `.ts`).
3. Next.js binds to `0.0.0.0:3000`.
4. The Route Handler at `app/api/health/route.ts` is registered.
5. When `GET /health` is received, the handler executes with no database calls and returns `{ "status": "ok" }` immediately.
6. Response is sent with HTTP 200 and `Content-Type: application/json`.

**Note:** The health endpoint must **not** open a database connection. It is intentionally dependency-free so it can signal liveness even when the database is unavailable.

---

### Inputs

- `GET /health` — no query parameters, no request body, no authentication headers required.

---

### Outputs

- **Success:** HTTP 200, `Content-Type: application/json`, body: `{ "status": "ok" }`

---

### Validation

- No input validation required for this endpoint — it accepts any GET request.
- The endpoint must respond in under 100 ms (non-functional requirement per PRD §7).
- The `next.config.js` file must use the `.js` extension (not `.ts`) to avoid Next.js 14 config-loading issues.
- `tsconfig.json` must enable `"strict": true`.

---

### Error States

| Scenario | HTTP Status | Error Code | Notes |
|----------|-------------|------------|-------|
| Server not started / port in use | N/A — connection refused | N/A | Operator must free port 3000 before starting |
| Next.js config file uses `.ts` extension on Next 14 | N/A — startup crash | N/A | Use `.js` extension; see Risks §PRD |
| `GET /health` receives wrong method (e.g. POST) | 405 | N/A | Next.js returns 405 Method Not Allowed automatically |

---

### API Surface (this feature)

| Method | Path | Auth | Request Body | Success Response |
|--------|------|------|--------------|-----------------|
| GET | `/health` | None | None | 200 `{ "status": "ok" }` |

Full spec: see `Y1-api.md` §Health.

---

### Schema Surface (this feature)

None. The health endpoint has no database dependency.

---
