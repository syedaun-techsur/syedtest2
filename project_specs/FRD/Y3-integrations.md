---

## External Integration Points

QuickNotes v1 has minimal external dependencies by design. This section catalogs all external systems the application touches, their contracts, and failure modes.

---

### §PostgreSQL Database

**Type:** Relational database  
**Used by:** F1 (schema & connectivity), F2 (all CRUD operations)  
**Connection:** Via `DATABASE_URL` environment variable  

| Property | Value |
|----------|-------|
| Protocol | PostgreSQL wire protocol (TCP) |
| Connection string format | `postgresql://user:password@host:port/dbname` |
| Client library | `pg` (node-postgres) or Prisma |
| Connection model | Singleton connection pool (shared across all Route Handler invocations) |
| Startup behaviour | Must be reachable before app serves requests; fail-fast if not |
| Migration strategy | Idempotent DDL (`CREATE TABLE IF NOT EXISTS`) run at startup; OR `prisma migrate deploy` / `prisma db push` |
| Sidecar mode | When `PIVOTA_DB_MODE=sidecar` is set, connect directly — same connection logic as normal mode in v1 |

**Failure modes:**
- Unreachable at startup → process exits with code 1 (see `Y2-errors.md` §Startup Errors).
- Unreachable at request time → Route Handler catches pg error and returns HTTP 500.

**Security note:** `DATABASE_URL` contains credentials. Never log the full URL; strip the password when logging connection errors.

---

### §Node.js Runtime

**Type:** JavaScript/TypeScript runtime  
**Version:** Node.js LTS (compatible with Next.js 14+)  
**Used by:** All features  

| Property | Value |
|----------|-------|
| Entry point | `npm run dev` (development) / `npm start` (production) |
| Port binding | `0.0.0.0:3000` |
| Environment | Reads `DATABASE_URL`, `PIVOTA_DB_MODE`, and standard Next.js env vars |

**No Docker dependency.** The app must run with `node` / `npm` directly. Docker or docker-compose must not be required at runtime.

---

### §Browser (Client-Side)

**Type:** Web browser (evergreen)  
**Supported browsers:** Chrome, Firefox, Safari, Edge (latest versions)  
**Used by:** F3, F4, F5 (UI features)  

| Property | Value |
|----------|-------|
| Protocol | HTTP (or HTTPS in production) |
| API communication | `fetch` API (native browser, no external HTTP client library required) |
| Rendering | Next.js SSR for initial HTML; React client-side hydration for interactivity |
| No native mobile app | Web only in v1 |

**Browser APIs used:**
- `fetch` — for all API calls to `/api/notes` and `/health`
- `window.confirm` — for delete confirmation prompt (v1; may be replaced with custom widget)
- `Date` — for relative timestamp computation

---

### §Out-of-Scope Integrations (v1)

The following external integrations are explicitly **not** implemented in v1:

| Integration | Status | Notes |
|-------------|--------|-------|
| Authentication / Identity Provider (e.g. Auth0, Azure AD) | Out of scope | Single-user, no auth |
| Email / notification service | Out of scope | No notifications in v1 |
| File storage (S3, local disk) | Out of scope | No file attachments in v1 |
| WebSocket / real-time sync | Out of scope | Single-user; no cross-tab sync |
| Docker / docker-compose | Out of scope — intentionally excluded | Runtime provisions DB natively |
| CDN / edge caching | Out of scope | Single Next.js process serves everything |

---
