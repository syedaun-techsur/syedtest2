# Y2: Accessibility Notes

**Applies to:** All screens  
**User Stories:** US-3.4 (Compose Box), US-5.5 (Edit/Delete Controls)  
**Standard:** WCAG 2.1 AA as baseline

---

## Summary of Requirements

QuickNotes has two explicitly specified accessibility requirements as acceptance criteria:
1. **US-3.4** — Compose box fully keyboard-accessible with proper label associations
2. **US-5.5** — Edit and Delete controls on note cards keyboard-accessible

All other accessibility notes below are strongly recommended to meet WCAG 2.1 AA and serve users who rely on assistive technology.

---

## 1. Label Associations (US-3.4, US-5.5)

Every form input must have an associated `<label>` element. Two valid patterns:

**Pattern A — Explicit `htmlFor` / `id` pairing (preferred):**
```html
<label htmlFor="body">Note</label>
<textarea id="body" placeholder="Write a note…"></textarea>
```

**Pattern B — Wrapping label:**
```html
<label>
  Note
  <textarea placeholder="Write a note…"></textarea>
</label>
```

**Required label associations:**

| Control | Label Text | Location |
|---|---|---|
| Compose box title input | "Title" | Compose box (US-3.4) |
| Compose box body textarea | "Note" | Compose box (US-3.4) |
| Edit mode title input | "Title" | Note card edit mode (US-5.5) |
| Edit mode body textarea | "Note" | Note card edit mode (US-5.5) |
| "Add Note" button | Button text is its own label | Compose box |
| "Save" button | Button text is its own label | Note card edit mode |
| "Cancel" button | Button text is its own label | Note card edit mode |
| "Edit" button | Should have `aria-label="Edit note: {title}"` | Note card (disambiguates multiple Edit buttons) |
| "Delete" button | Should have `aria-label="Delete note: {title}"` | Note card (disambiguates multiple Delete buttons) |

**Multiple Edit/Delete buttons:** Because the page may have many "Edit" buttons (one per card), each button should have a unique accessible label via `aria-label` that includes the note title (or "Untitled"). This allows screen reader users to distinguish them.

```html
<button aria-label="Edit note: My Meeting Note">Edit</button>
<button aria-label="Delete note: My Meeting Note">Delete</button>
```

---

## 2. Keyboard Navigation (US-3.4, US-5.5)

### Tab Order — Compose Box
1. Title `<input>` (optional, first Tab stop)
2. Body `<textarea>` (required, second Tab stop)
3. "Add Note" `<button>` (third Tab stop)

### Tab Order — Note Card (Read View)
Within each card, the Tab order should be logical:
1. "Edit" button
2. "Delete" button

### Tab Order — Note Card (Edit Mode)
1. Title `<input>` (pre-filled)
2. Body `<textarea>` (pre-filled)
3. "Save" `<button>`
4. "Cancel" `<button>`

### Keyboard Activation
- All buttons (`<button>` elements) are activatable via **Enter** and **Space** when focused (native HTML behaviour)
- Do not use `<div>` or `<span>` as buttons — use `<button>` elements (native keyboard behaviour)
- The `disabled` attribute on the "Add Note" and "Save" buttons must prevent keyboard activation, not just mouse click

### Focus Management
- When a note card enters edit mode, focus should ideally move to the Title input (or Body textarea) of that card
- When a note card exits edit mode (Save success or Cancel), focus returns to the "Edit" button on the card
- When a note is deleted, focus should move to the next card's "Delete" or "Edit" button, or to the compose box if no cards remain

---

## 3. ARIA Roles and Live Regions

### Error Messages
All inline error messages should use `role="alert"` or `aria-live="polite"` so screen readers announce them when they appear:

```html
<!-- Error zone — announced automatically when content changes -->
<p role="alert" aria-live="polite">
  Failed to save note. Please try again.
</p>
```

Use `role="alert"` for errors (immediate announcement) and `aria-live="polite"` for non-urgent status updates.

### Note List Loading State
```html
<div aria-live="polite" aria-label="Note list status">
  Loading notes…
</div>
```

### Empty State
```html
<p>No notes yet — add your first one above.</p>
```
This requires no special ARIA — it is plain text content and will be read by screen readers.

### Note Cards
```html
<article aria-label="Note: {title or 'Untitled'}">
  ...card contents...
</article>
```

Using `<article>` is semantically appropriate for self-contained note cards and provides screen reader context.

---

## 4. Semantic HTML

| Element | Use Case |
|---|---|
| `<h1>` | Page title "QuickNotes" — one per page |
| `<main>` | Main content area wrapping compose box + note list |
| `<form>` | Compose box (wraps inputs and submit button) |
| `<label>` | All form field labels |
| `<button>` | All interactive controls (never `<div onClick>`) |
| `<textarea>` | Multi-line text inputs (body fields) |
| `<input type="text">` | Single-line text inputs (title fields) |
| `<time dateTime="...">` | All relative timestamps (US-4.1) |
| `<article>` | Each note card |
| `<ul>` / `<li>` | Note card list container (optional; provides list semantics) |

**`<time>` element requirement (US-4.1):**
```html
<time dateTime="2026-07-04T12:00:00.000Z">2 minutes ago</time>
```
The `dateTime` attribute contains the machine-readable ISO 8601 UTC timestamp; the display text is the human-readable relative string.

---

## 5. Colour Contrast

- **Body text on background:** Minimum 4.5:1 contrast ratio (WCAG 2.1 AA for normal text)
- **"Untitled" fallback text:** If displayed in italic/muted colour, must maintain ≥ 4.5:1 contrast (or ≥ 3:1 if large text)
- **Disabled button:** May have reduced contrast (WCAG exception for disabled elements), but should still be visually distinguishable
- **Error messages:** Red/orange error text must maintain ≥ 4.5:1 against its background
- **Timestamp text:** If displayed in a lighter/smaller style, must still meet minimum contrast ratios

**Do not rely on colour alone** to communicate state (e.g., disabled state should not be conveyed by colour only — the native `disabled` attribute communicates this to assistive technology).

---

## 6. Server-Side Render Requirement (US-3.4)

The compose box must be present in the **initial server-rendered HTML** (Next.js SSR), not only after client-side JavaScript hydration. This ensures:
- Screen readers can access the compose box before JS loads
- Users with JS disabled or slow JS execution can see the form structure
- Search engine crawlers see the form (relevant if the app is ever made public)

The note list data is fetched client-side (React state after hydration), which is acceptable — the structure (list container, empty state placeholder) may be SSR'd.

---

## 7. Motion and Animation

- QuickNotes has minimal required animation. Card transitions (read → edit mode) should be **instant or very fast** (< 150ms) to avoid motion sickness
- Avoid `prefers-reduced-motion: reduce` violations — if any CSS transitions are added, wrap them in a media query respecting the user's motion preference:

```css
@media (prefers-reduced-motion: no-preference) {
  .card-transition { transition: all 0.15s ease; }
}
```

---

## Accessibility Checklist

- [ ] All form inputs have associated `<label>` elements (US-3.4, US-5.5)
- [ ] Multiple "Edit"/"Delete" buttons have unique `aria-label` values including note title
- [ ] Tab order is logical within compose box and note cards (US-3.4, US-5.5)
- [ ] All buttons are `<button>` elements (not `<div>`/`<span>`)
- [ ] "Add Note" and "Save" disabled state uses native HTML `disabled` attribute
- [ ] Error zones use `role="alert"` or `aria-live="polite"`
- [ ] Timestamps use `<time dateTime="...">` with ISO 8601 UTC value (US-4.1)
- [ ] Colour contrast meets 4.5:1 for all body text
- [ ] Compose box is in SSR initial HTML (US-3.4)
- [ ] Focus management is handled on edit mode enter/exit and card deletion
