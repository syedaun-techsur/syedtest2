# Screen-02: Note Card — Edit Mode

**Purpose:** Inline editing of a single note card, in place within the note list. No page navigation.  
**User Stories:** US-5.1, US-5.2, US-5.3, US-5.5  
**Feature:** F5  
**Location:** Within the Note List (Screen-01), replacing the read view of the targeted card

---

## Layout — Edit Mode (Card Expanded)

```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (above — unchanged, still usable)              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── NOTE CARD (EDIT MODE) ──────────────────────────────┐ │
│  │                                                        │ │
│  │  <label for="edit-title-{id}">Title</label>            │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │  My Meeting Note                                 │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │  <input type="text" id="edit-title-{id}"               │ │
│  │   value="My Meeting Note">                             │ │
│  │                                                        │ │
│  │  <label for="edit-body-{id}">Note</label>              │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │                                                  │  │ │
│  │  │  Follow up with Sam re: pricing by end of week.  │  │ │
│  │  │                                                  │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │  <textarea id="edit-body-{id}">                        │ │
│  │                                                        │ │
│  │  [ Save ]  [ Cancel ]                                  │ │
│  │  (Save disabled when body empty)                       │ │
│  │                                                        │ │
│  │  (error zone — hidden by default)                      │ │
│  │  ⚠ Failed to save. Please try again.                   │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌─── OTHER NOTE CARDS (read view, unchanged) ────────────┐ │
│  │  ...                                                   │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Read View vs Edit Mode Comparison

| Element | Read View | Edit Mode |
|---|---|---|
| Title display | Plain text (or "Untitled" italic) | `<input type="text">` pre-filled with `note.title` |
| Body display | Plain text | `<textarea>` pre-filled with `note.body` |
| Timestamp | `<time>` relative string | Hidden during edit mode |
| Action buttons | "Edit" + "Delete" | "Save" + "Cancel" |
| Error zone | Not present | Shown on PUT failure |

---

## Interactive Elements (Edit Mode)

| Element | Type | Behaviour |
|---|---|---|
| Title input | `<input type="text" id="edit-title-{id}">` | Pre-filled with `note.title` or `""` if null; optional |
| Body textarea | `<textarea id="edit-body-{id}">` | Pre-filled with `note.body`; required (drives Save disabled state) |
| Save button | `<button>` | Disabled (native `disabled` attr) when `body.trim() === ""`; calls PUT on click |
| Cancel button | `<button>` | Always enabled; reverts card to read view; no API call |
| Inline error zone | `<p role="alert">` | Shown on PUT failure; hidden by default |

---

## States (Edit Mode)

| State | Appearance | User Feedback |
|---|---|---|
| **Initial (just opened)** | Inputs pre-filled; Save enabled if body non-empty | Ready to edit |
| **Body cleared by user** | Save button becomes disabled | Cannot save empty body |
| **Saving** | Save button shows "Saving…"; both buttons disabled | In-progress signal; prevents double-submit |
| **Save success** | Card exits edit mode; read view shows updated content | Immediate in-place update |
| **Save error — generic** | Card stays in edit mode; error message shown | "Failed to save. Please try again." |
| **Save error — 404** | Card stays in edit mode; specific error message | "Note not found. It may have been deleted." |
| **Cancelled** | Card reverts to read view; original values restored | No changes persisted |

---

## Validation Rules (Edit Mode)

| Rule | Behaviour |
|---|---|
| Body must be non-empty after trim | Save button has native `disabled` attribute |
| Title is optional | Empty string normalised to `null` on save |
| Cancel restores exact original values | Do not persist trimmed or modified intermediates |
| Only one card in edit mode at a time | Other cards remain in read view (US-5.1) |

---

## DOM / Accessibility Requirements (Edit Mode)

```html
<!-- Edit mode card structure (US-5.5) -->
<article aria-label="Editing note: {title}">
  <label for="edit-title-{id}">Title</label>
  <input type="text" id="edit-title-{id}" value="{note.title}" />

  <label for="edit-body-{id}">Note</label>
  <textarea id="edit-body-{id}">{note.body}</textarea>

  <button type="submit" disabled="{bodyEmpty}">Save</button>
  <button type="button">Cancel</button>

  <!-- Error zone — conditionally rendered -->
  <p role="alert" aria-live="polite">{errorMessage}</p>
</article>
```

- Tab order within card: Title input → Body textarea → Save → Cancel (US-5.5)
- Save and Cancel activatable via Enter/Space when focused (US-5.5)
- Focus is not trapped — user can Tab out of the card (accessibility best practice)
- `id` attributes on inputs must be unique per card (include note ID in `id`)

---

## Error Messages

| Scenario | Error Message | Card Behaviour |
|---|---|---|
| PUT returns 500 or network failure | "Failed to save. Please try again." | Stays in edit mode |
| PUT returns 404 (note gone) | "Note not found. It may have been deleted." | Stays in edit mode |
| PUT returns 400 (body empty — server guard) | "Failed to save. Please try again." | Stays in edit mode (should not occur if client validates) |
