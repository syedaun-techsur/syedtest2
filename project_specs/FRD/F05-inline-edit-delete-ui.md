---

## F05: Single-Page UI — Inline Edit & Delete

**Description:** Each note card in the list (F4) exposes "Edit" and "Delete" controls. The edit control transitions the card into an inline editing mode with pre-populated fields; saving calls `PUT /api/notes/:id` and updates the card in place. The delete control prompts for confirmation and then calls `DELETE /api/notes/:id`, removing the card from the list. No page navigation is required for either action.

**Priority:** P1 — High. Required for full CRUD in the UI; layered after the list (F4) is in place.

**Depends On:** F2 (`PUT /api/notes/:id` and `DELETE /api/notes/:id`), F4 (note list hosts the cards).

---

### Terminology

- **Read view:** The default rendered state of a note card — showing title, body, timestamp, and the Edit/Delete control buttons.
- **Edit mode:** The inline editing state of a note card — showing editable title input and body textarea pre-filled with current values, plus "Save" and "Cancel" buttons.
- **Confirmation prompt:** A synchronous browser dialog (`window.confirm("Are you sure?")`) or an inline confirmation widget displayed before executing a delete.
- **Optimistic removal:** Removing a note card from the list immediately on delete confirmation, before the API response, to improve perceived responsiveness. The card is restored if the API call fails.

---

### Sub-features

- "Edit" button on each note card in read view
- Inline transition to edit mode (title input + body textarea pre-populated)
- "Save" button in edit mode — calls `PUT /api/notes/:id`
- "Cancel" button in edit mode — reverts to read view without saving
- "Delete" button on each note card in read view
- Confirmation prompt before delete
- `DELETE /api/notes/:id` call on confirm; card removal from list on success
- Keyboard accessibility for all controls (Tab-navigable)

---

### Process — Edit Flow

1. Note card is in read view. "Edit" and "Delete" buttons are visible.
2. User clicks (or keyboard-activates) "Edit".
3. Card transitions to edit mode:
   - Title field: `<input type="text">` pre-filled with `note.title` (or `""` if null).
   - Body field: `<textarea>` pre-filled with `note.body`.
   - "Save" button and "Cancel" button replace the "Edit" / "Delete" buttons.
4. User modifies title and/or body.
5. "Save" button is disabled when `body.trim() === ""`.
6. User clicks "Save":
   a. Client trims values: `title = title.trim() || null`, `body = body.trim()`.
   b. Calls `PUT /api/notes/:id` with `{ title, body }`.
   c. While awaiting response: "Save" button shows loading state (e.g. "Saving…"), both buttons temporarily disabled.
   d. On success (HTTP 200):
      - Card exits edit mode and returns to read view.
      - Card displays updated title, body, and timestamp from the API response.
      - No full list re-fetch required — card state updated locally from PUT response.
   e. On error (non-200 response):
      - Card remains in edit mode.
      - Inline error displayed near the edit form (e.g. "Failed to save. Please try again.").
7. User clicks "Cancel" (at any point during edit mode):
   - Card reverts to read view with original values (no API call made).
   - Edits are discarded.

---

### Process — Delete Flow

1. Note card is in read view. "Delete" button is visible.
2. User clicks (or keyboard-activates) "Delete".
3. A confirmation prompt is shown: `"Are you sure you want to delete this note?"`
   - Implementation: `window.confirm(...)` is acceptable for v1. A custom inline widget is optional.
4. If user cancels the confirmation → no action; card remains in read view.
5. If user confirms:
   a. Card may be optimistically hidden from the list immediately (optional).
   b. Client calls `DELETE /api/notes/:id`.
   c. On success (HTTP 204):
      - Card is permanently removed from the list.
      - If the deleted note was the last one, the empty state message appears.
   d. On error (non-204 response or network failure):
      - If optimistically hidden, card is restored to the list.
      - Inline or toast error: "Failed to delete note. Please try again."

---

### Inputs

**Edit mode — user-facing fields:**
- `title` (text input, optional): Pre-filled with current `note.title` or `""`.
- `body` (textarea, required): Pre-filled with current `note.body`. Cannot be empty on save.

**Submitted to API (PUT):**
- `title` (string | null): `title.trim()` or `null` if empty.
- `body` (string): `body.trim()`. Non-empty (enforced before call).

**Delete confirmation:**
- Implicit: user accepts the browser `confirm()` dialog (or custom UI equivalent).

---

### Outputs

- **Edit success:** Updated note card in read view with new title, body, and (optionally refreshed) timestamp.
- **Edit cancel:** Card unchanged, returned to read view.
- **Delete success:** Card removed from DOM; empty state shown if no notes remain.
- **Edit/Delete error:** Inline error message; card state preserved.

---

### Validation

- Body field in edit mode must be non-empty after trimming before "Save" is enabled (same rule as F3 compose box).
- Title in edit mode is optional; normalised to `null` if empty.
- Each note card's edit/delete controls must be keyboard-reachable via Tab key and activatable via Enter/Space.
- The "Save" button must carry the native `disabled` attribute when body is empty.
- Cancelling edit must restore the exact original values (do not persist trimmed or modified intermediate state).
- Only one note card may be in edit mode at a time. If the user opens edit on one card, other cards remain in read view (or, alternatively, opening edit on a new card auto-cancels the previous edit — either is acceptable; specify behaviour in implementation).

---

### Error States

| Scenario | User-Visible Behaviour | Technical Detail |
|----------|----------------------|-----------------|
| `PUT` returns 404 (note deleted by another session) | "Note not found. It may have been deleted." | Single-user app; edge case |
| `PUT` returns 400 (body empty — server-side guard) | "body is required." | Should not occur if client validates |
| `PUT` returns 500 | "Failed to save. Please try again." | Card stays in edit mode |
| `DELETE` returns 404 | "Note not found." Remove card anyway. | Card already gone from DB |
| `DELETE` returns 500 | "Failed to delete. Please try again." | Restore card if optimistically removed |
| Network error on PUT/DELETE | Same as 500 behaviour above | `fetch` throws |
| User cancels delete confirmation | No action, card remains | `confirm()` returns `false` |
| Body empty in edit mode | "Save" button disabled | Client-side only guard |

---

### API Surface (this feature)

| Method | Path | Called When | Expected Response |
|--------|------|------------|-----------------|
| PUT | `/api/notes/:id` | User clicks "Save" in edit mode | 200 with updated note |
| DELETE | `/api/notes/:id` | User confirms delete | 204 No Content |

Full schemas: see `Y1-api.md` §Notes — PUT and DELETE.

---

### Schema Surface (this feature)

No direct database access. Reads/writes through F2 API only.

---
