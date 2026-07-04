---

## F04: Single-Page UI — Note List

**Description:** Below the compose box, a scrollable list of all persisted notes is displayed, ordered newest first. The list is fetched from `GET /api/notes` on initial page load and refreshed after every create, edit, or delete action without a full page reload. Each note card shows the title, body, and a relative timestamp. When no notes exist, a friendly empty-state message is displayed.

**Priority:** P0 — Critical. Primary read surface for all persisted notes.

**Depends On:** F2 (`GET /api/notes` must be available), F3 (list refresh is triggered by compose box after successful create).

---

### Terminology

- **Note card:** A UI element representing a single note, displaying its title, body text, and relative timestamp.
- **Relative timestamp:** A human-readable time-since string derived from `created_at` (e.g. "just now", "2 minutes ago", "yesterday", "3 days ago"). Computed client-side.
- **Empty state:** The UI condition when `GET /api/notes` returns an empty array; displays "No notes yet — add your first one above."
- **List refresh:** Re-calling `GET /api/notes` and re-rendering the list in place without a full browser navigation or page reload.

---

### Sub-features

- Initial note list fetch from `GET /api/notes` on page load
- Rendering note cards with title, body, and relative timestamp
- "Untitled" fallback for notes with null/empty title
- Empty state message when no notes exist
- List re-renders after create (F3), edit (F5), or delete (F5) without page reload
- Notes ordered newest first (as returned by the API — no client-side re-sorting required)

---

### Process

1. Page component mounts (client-side after SSR hydration or as a client component).
2. `GET /api/notes` is called.
3. While fetching, a loading indicator may be shown (e.g. "Loading notes…") or the list area is left empty.
4. On success (HTTP 200):
   a. If the response array is empty → render empty state: `<p>No notes yet — add your first one above.</p>`.
   b. If the array has items → render one note card per item in the order returned (newest first).
5. Each note card renders:
   - **Title line:** Display `note.title` if non-null and non-empty; otherwise display `"Untitled"` (visually distinguished, e.g. italic or muted colour).
   - **Body:** Display `note.body` as plain text (no Markdown rendering in v1).
   - **Timestamp:** Display relative time derived from `note.created_at` (e.g. using a library like `date-fns` `formatDistanceToNow` or a hand-rolled helper).
   - **Edit control:** "Edit" button (see F5).
   - **Delete control:** "Delete" button (see F5).
6. When F3 (compose box) successfully creates a note, it signals the list to re-fetch → repeat from step 2.
7. When F5 (inline edit/delete) modifies or removes a note, it signals the list to re-fetch → repeat from step 2. Alternatively, for delete, the card may be removed optimistically from local state.

---

### Inputs

- `GET /api/notes` response: JSON array of note objects (see `Y1-api.md` §Notes — GET collection).
- Refresh signals: emitted by F3 after successful POST, and by F5 after successful PUT or DELETE.

---

### Outputs

- **Non-empty list:** One note card per note, in newest-first order.
- **Empty state:** Single message: "No notes yet — add your first one above."
- **Loading state:** Transient indicator while fetching (optional but recommended for UX).
- **Error state:** If `GET /api/notes` fails, display an inline error message (e.g. "Failed to load notes. Refresh to try again.").

---

### Validation

- The list must display notes in the order returned by the API (`ORDER BY created_at DESC`). No client-side re-sorting.
- `note.title` of `null`, `""`, or whitespace-only must all render as "Untitled". Do not display a blank title area.
- Relative timestamps must update if the page remains open long enough to be meaningful (a simple static render on fetch is acceptable for v1; live ticking is optional).
- The empty state message must match exactly: `"No notes yet — add your first one above."`
- List updates must not cause a full page reload; React state update and re-render is the expected mechanism.
- Accessibility: each note card must be readable by screen readers; timestamp rendered with a machine-readable `datetime` attribute on a `<time>` element (e.g. `<time dateTime={note.created_at}>2 minutes ago</time>`).

---

### Error States

| Scenario | User-Visible Behaviour | Technical Detail |
|----------|----------------------|-----------------|
| `GET /api/notes` returns empty array | Empty state message displayed | Normal condition, not an error |
| `GET /api/notes` returns 500 | "Failed to load notes. Refresh to try again." | List area shows error, compose box still usable |
| Network error on initial load | Same error message as 500 | `fetch` throws |
| List refresh fails after create/edit/delete | Previous list state retained; optional retry toast | Non-blocking; persisted data is safe |

---

### API Surface (this feature)

| Method | Path | Called When | Expected Response |
|--------|------|------------|-----------------|
| GET | `/api/notes` | Page load and after any mutation | 200 with note array |

Full schema: see `Y1-api.md` §Notes — GET collection.

---

### Schema Surface (this feature)

No direct database access. Reads through F2 API only.

---
