# Screen-00: Compose Box

**Purpose:** Primary capture surface — always visible at the top of the page. Allows users to write and submit a new note in under 5 seconds.  
**User Stories:** US-3.1, US-3.2, US-3.3, US-3.4  
**Feature:** F3  
**Location:** Top zone of the single root page (`/`), above the fold on all viewports

---

## Layout

```
┌─────────────────────────────────────────────────────────────┐
│  <h1> QuickNotes </h1>                                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── COMPOSE BOX ───────────────────────────────────────┐  │
│  │                                                       │  │
│  │  <label for="title">Title</label>                     │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  Title (optional)                               │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │  <input type="text" id="title" placeholder="Title     │  │
│  │   (optional)" — not required>                         │  │
│  │                                                       │  │
│  │  <label for="body">Note</label>                       │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │                                                 │  │  │
│  │  │  Write a note…                                  │  │  │
│  │  │                                                 │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │  <textarea id="body" placeholder="Write a note…">     │  │
│  │                                                       │  │
│  │  [ Add Note ]  ← <button disabled> when body empty    │  │
│  │                                                       │  │
│  │  (error zone — hidden by default)                     │  │
│  │  ⚠ Failed to save note. Please try again.             │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  NOTE LIST (see Screen-01)                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Information Hierarchy

| Priority | Content | Placement | Notes |
|---|---|---|---|
| Primary | Body textarea | Centre, large touch target | Required field; receives focus first |
| Secondary | "Add Note" button | Below body textarea | Disabled state when body empty |
| Tertiary | Title input | Above body textarea | Optional; not required |
| Contextual | Inline error message | Below button | Only visible on failure |
| Page-level | "QuickNotes" `<h1>` | Page header | Identifies app; not repeated |

---

## Interactive Elements

| Element | Type | Behaviour |
|---|---|---|
| Title input | `<input type="text">` | Optional; placeholder "Title (optional)"; associated `<label htmlFor="title">` |
| Body textarea | `<textarea>` | Required; placeholder "Write a note…"; associated `<label htmlFor="body">`; drives button disabled state |
| Add Note button | `<button>` | Disabled (native `disabled` attr) when `body.trim() === ""`; enabled when body has content |
| Inline error zone | `<p>` or `<div>` | Hidden by default; shown on POST failure; message: "Failed to save note. Please try again." |

---

## States

| State | Visual Appearance | User Feedback |
|---|---|---|
| **Default (empty)** | Title input empty, body textarea empty, "Add Note" button disabled (greyed out) | Button visually de-emphasised; not clickable |
| **Body has content** | Body textarea has text, "Add Note" button enabled (normal appearance) | Button becomes interactive |
| **Loading / Saving** | "Add Note" button shows "Saving…" text, button disabled temporarily | Prevents double-submit; clear in-progress signal |
| **Success** | Both fields cleared, button returns to disabled | Note appears in list below — visual confirmation |
| **Error** | Fields preserved with user's input; red/warning inline message below button | "Failed to save note. Please try again." |

---

## Validation Rules

| Rule | Client Behaviour | Server Fallback |
|---|---|---|
| Body required | Button has native `disabled` attribute; also guarded client-side before fetch | API returns 400 `{ "error": "body is required" }` |
| Whitespace-only body | `body.trim() === ""` → button stays disabled | Same 400 response |
| Title optional | No restriction; empty string sent as `null` to API | Stored as NULL |
| Form not cleared on error | Values preserved for retry | N/A |

---

## DOM / Accessibility Requirements

```html
<!-- Required structure (US-3.4) -->
<form>
  <label for="title">Title</label>
  <input type="text" id="title" placeholder="Title (optional)" />

  <label for="body">Note</label>
  <textarea id="body" placeholder="Write a note…"></textarea>

  <button type="submit" disabled>Add Note</button>

  <!-- Error zone — conditionally rendered -->
  <p role="alert" aria-live="polite">Failed to save note. Please try again.</p>
</form>
```

- Tab order: Title input → Body textarea → Add Note button (US-3.4)
- "Add Note" activatable via Enter or Space when focused (US-3.4)
- Compose box in SSR initial HTML (not client-render-only, US-3.4)
- Error zone uses `role="alert"` or `aria-live="polite"` so screen readers announce it
