---

## F03: Single-Page UI — Compose Box

**Description:** The compose box is the primary capture surface, always visible at the top of the single page. It provides an optional title input and a required body textarea so users can write and submit a new note in under 5 seconds from app open. On submit, it calls `POST /api/notes`, clears itself on success, and triggers a refresh of the note list (F4).

**Priority:** P0 — Critical. Core capture flow.

**Depends On:** F2 (`POST /api/notes` must be available). F4 (list refresh is coordinated with the note list component).

---

### Terminology

- **Compose box:** The form section at the top of the page containing the title input, body textarea, and submit button.
- **Disabled state:** The "Add Note" button rendered with `disabled` attribute and visually de-emphasised when the body is empty or whitespace-only.
- **Form clear:** Resetting both input fields to empty strings after a successful submission.
- **List refresh:** Re-fetching `GET /api/notes` and re-rendering the note list to include the newly created note.

---

### Sub-features

- Page header displaying the application title "QuickNotes"
- Optional title text input with placeholder "Title (optional)"
- Required body textarea with placeholder "Write a note…"
- "Add Note" submit button with disabled logic
- `POST /api/notes` call on submit with loading and error feedback
- Form clear on success
- Note list refresh trigger after successful submission

---

### Process

1. Page renders with compose box visible at the top.
2. Page header `<h1>` (or equivalent) displays "QuickNotes".
3. Title input is rendered as `<input type="text">` with `placeholder="Title (optional)"`, associated `<label>` ("Title"), and not required.
4. Body textarea is rendered as `<textarea>` with `placeholder="Write a note…"`, associated `<label>` ("Note"), and is functionally required.
5. "Add Note" `<button>` is rendered with `disabled` attribute when `body.trim() === ""` (empty or whitespace-only); enabled otherwise.
6. User types in the body field → button becomes enabled.
7. User clicks "Add Note" (or submits via keyboard).
8. Client trims `body` value; if empty, does not submit (button is disabled, but also guard client-side).
9. Client calls `POST /api/notes` with `{ title: titleValue.trim() || null, body: body.trim() }`.
10. While awaiting response, the button may show a loading indicator (e.g. "Saving…") and be temporarily disabled.
11. On success (HTTP 201):
    - Clear title input to `""`.
    - Clear body textarea to `""`.
    - Trigger note list refresh (re-fetch `GET /api/notes`).
12. On error (non-201 response):
    - Display an inline error message near the form (e.g. "Failed to save note. Please try again.").
    - Do not clear the form so the user can retry.

---

### Inputs

**User-facing fields:**
- `title` (text input, optional): Free-form text. Max display length unconstrained in v1.
- `body` (textarea, required): Free-form text. Must contain at least one non-whitespace character before submission is allowed.

**Submitted to API:**
- `title` (string | null): `title.trim()` or `null` if empty after trim.
- `body` (string): `body.trim()`. Non-empty (enforced before call).

---

### Outputs

- **Success path:** Note created, form cleared, note list updated.
- **Error path:** Inline error message displayed; form preserved.
- **Accessibility output:** All interactive elements reachable via Tab; labels associated via `htmlFor` / `id` pairing or `aria-label`.

---

### Validation

- Body must be non-empty after trimming before submission is allowed (client-side and API-level).
- Title is always optional; empty string submitted as `null` to the API.
- The `<button>` must carry the native HTML `disabled` attribute (not just CSS styling) so keyboard and assistive technology users also cannot activate it.
- Each form field must have an associated `<label>` element (explicit `htmlFor`/`id` association or wrapping label).
- The compose box must be rendered server-side in the initial HTML (Next.js SSR); form interactivity is handled client-side (React state).

---

### Error States

| Scenario | User-Visible Behaviour | Technical Detail |
|----------|----------------------|-----------------|
| Body is empty | "Add Note" button disabled; cannot submit | `body.trim() === ""` check |
| `POST /api/notes` returns 400 | Inline error: "Failed to save note. Please try again." | Form not cleared |
| `POST /api/notes` returns 500 | Inline error: "Failed to save note. Please try again." | Form not cleared |
| Network error (fetch throws) | Inline error: "Failed to save note. Please try again." | Form not cleared |
| `POST /api/notes` succeeds but list refresh fails | Note created; list may be stale; silent retry or page reload | Non-blocking; note is persisted |

---

### API Surface (this feature)

| Method | Path | Called When | Expected Response |
|--------|------|------------|-----------------|
| POST | `/api/notes` | User clicks "Add Note" | 201 with created note |

Full schema: see `Y1-api.md` §Notes — POST.

---

### Schema Surface (this feature)

No direct database access. Reads/writes through F2 API only.

---
