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
