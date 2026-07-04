# Y1: Responsive Considerations

**Applies to:** All screens  
**User Stories:** US-3.1, US-3.4, US-4.1, US-5.5  
**Browser targets:** Chrome, Firefox, Safari, Edge (latest evergreen, PRD §6)

---

## Layout Philosophy

QuickNotes is a **single-column, single-page** application. The layout is inherently responsive by nature — there is no complex grid, sidebar, or multi-panel layout to manage. The primary responsive concern is ensuring the compose box remains **above the fold** on all viewports, so Alex can start typing immediately without scrolling.

---

## Breakpoint Definitions

| Breakpoint | Label | Width Range |
|---|---|---|
| Mobile | sm | < 640px |
| Tablet | md | 640px – 1023px |
| Desktop | lg | ≥ 1024px |

---

## Desktop (≥ 1024px)

```
┌────────────────────────────────────────────────────────────────┐
│  QuickNotes                                                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌── COMPOSE BOX (max-width: ~720px, centred) ──────────────┐  │
│  │  Title input (full width of box)                         │  │
│  │  Body textarea (full width, ~4 rows min)                 │  │
│  │  [Add Note] button (right-aligned or left-aligned)       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                │
│  ┌── NOTE LIST (max-width: ~720px, centred) ────────────────┐  │
│  │  ┌── Note Card ────────────────────────────────────────┐ │  │
│  │  │  Title                              Timestamp       │ │  │
│  │  │  Body text (full width of card)                     │ │  │
│  │  │                              [Edit]  [Delete]       │ │  │
│  │  └────────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**Desktop considerations:**
- Page content is constrained to a readable max-width (~720px) and centred horizontally
- Compose box and note list align in a single column
- Body textarea has comfortable height (~4–6 rows minimum) — no forced scroll to type
- "Add Note" button is clearly visible without scrolling on a standard 1080p screen

---

## Tablet (640px – 1023px)

```
┌──────────────────────────────────────────────────────┐
│  QuickNotes                                          │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌── COMPOSE BOX (full width, ~16px padding) ─────┐  │
│  │  Title input                                   │  │
│  │  Body textarea (~3–4 rows)                     │  │
│  │  [Add Note]                                    │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌── NOTE LIST (full width) ──────────────────────┐  │
│  │  ┌── Note Card ────────────────────────────┐   │  │
│  │  │  Title                 Timestamp        │   │  │
│  │  │  Body text                              │   │  │
│  │  │                    [Edit]  [Delete]     │   │  │
│  │  └────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────┘
```

**Tablet considerations:**
- Layout remains single-column; no structural changes from desktop
- Content fills available width with appropriate horizontal padding (~16px per side)
- Compose box must still be fully visible above the fold on a typical 768px portrait tablet
- Touch targets for buttons should be at minimum 44×44px (WCAG 2.5.5)

---

## Mobile (< 640px)

```
┌──────────────────────────────────────────┐
│  QuickNotes                              │
├──────────────────────────────────────────┤
│                                          │
│  ┌── COMPOSE BOX (full width) ─────────┐ │
│  │  Title input (full width)           │ │
│  │  Body textarea (~3 rows)            │ │
│  │  [Add Note]  ← full-width button    │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌── NOTE LIST ────────────────────────┐ │
│  │  ┌── Note Card ──────────────────┐  │ │
│  │  │  Title         Timestamp      │  │ │
│  │  │  Body text                    │  │ │
│  │  │  [Edit]  [Delete]             │  │ │
│  │  └──────────────────────────────┘  │ │
│  └────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

**Mobile considerations:**
- Compose box must be above the fold on a typical 375px-wide mobile viewport
- Body textarea minimum height: ~3 rows (enough to see the placeholder and start typing)
- "Add Note" button: full-width or near-full-width for easy thumb access
- Note card title and timestamp may stack vertically if space is tight (title on top row, timestamp below or right)
- "Edit" and "Delete" buttons must remain clearly visible and tappable (min 44px touch target)
- No hover states required on mobile — controls must be always visible (JRN-01.3: no hover-only controls)

---

## Critical Cross-Breakpoint Rules

| Rule | Reason |
|---|---|
| Compose box is ALWAYS above the fold | JRN-01.1 Orient stage: Alex cannot scroll to find it |
| Edit and Delete buttons are ALWAYS visible (not hover-only) | JRN-01.3: hover-only controls are a pain point; mobile has no hover |
| Touch targets ≥ 44×44px | WCAG 2.5.5 minimum; prevents accidental taps |
| No horizontal scroll | Single-column layout prevents overflow |
| Inline error messages stack below buttons | Not floated or overlaid — must not obscure content |

---

## Styling Approach

Per PRD §4: Tailwind CSS or CSS Modules are acceptable. The layout can be achieved with minimal styling:

```
Suggested Tailwind class pattern:
- Page container: max-w-2xl mx-auto px-4
- Section (compose/list): space-y-4
- Note card: rounded-lg border p-4
- Button: min-h-[44px] px-4 py-2
- Textarea: w-full min-h-[80px] resize-y
```

No complex grid framework required. A simple flexbox column layout suffices.
