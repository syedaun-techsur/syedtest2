# Screen-03: Delete Confirmation

**Purpose:** Confirm user intent before permanently deleting a note. Prevents accidental data loss.  
**User Stories:** US-5.4, US-4.2  
**Feature:** F5  
**Location:** Triggered from a note card's "Delete" button; appears as a browser dialog or inline prompt

---

## Confirmation Prompt — v1 Implementation (`window.confirm`)

```
┌─────────────────────────────────────────────────────────────┐
│  BROWSER DIALOG (native)                                    │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  Are you sure you want to delete this note?           │  │
│  │                                                       │  │
│  │                    [  OK  ]   [ Cancel ]              │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

- **Prompt text (exact):** "Are you sure you want to delete this note?" (US-5.4)
- **Implementation:** `window.confirm(...)` is acceptable for v1
- **OK:** Proceeds with `DELETE /api/notes/:id`
- **Cancel:** No action; card unchanged in read view

---

## After Confirmation — Visual States

### Deleting (in-progress)
```
┌─────────────────────────────────────────────────────────────┐
│  NOTE LIST                                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  (Card may be optimistically hidden / faded out)     │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Other cards remain visible and interactive          │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Delete Success — Last Note Remaining
```
┌─────────────────────────────────────────────────────────────┐
│  COMPOSE BOX (unchanged)                                    │
├─────────────────────────────────────────────────────────────┤
│  NOTE LIST                                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  No notes yet — add your first one above.            │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Delete Error (Non-204 or Network Failure)
```
┌─────────────────────────────────────────────────────────────┐
│  NOTE LIST                                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  My Meeting Note           2 minutes ago              │   │
│  │  Follow up with Sam re: pricing by end of week.       │   │
│  │                             [Edit]  [Delete]          │   │
│  └──────────────────────────────────────────────────────┘   │
│  ⚠ Failed to delete note. Please try again.                 │
│  (toast or inline message near the card)                    │
└─────────────────────────────────────────────────────────────┘
```

---

## States

| State | Appearance | Trigger |
|---|---|---|
| **Confirmation pending** | Browser `confirm()` dialog overlays page | User clicks "Delete" |
| **User cancels** | Dialog dismissed; card unchanged in read view | User clicks Cancel in dialog |
| **Deleting (optimistic)** | Card may be faded/hidden; other cards normal | User clicks OK in dialog |
| **Delete success** | Card removed from DOM; empty state if last | `DELETE` returns 204 |
| **Delete error** | Card restored (if hidden); inline error shown | `DELETE` returns non-204 or network fails |

---

## Error Messages

| Scenario | Message | Placement |
|---|---|---|
| DELETE returns non-204 or network error | "Failed to delete note. Please try again." | Inline near card or toast notification |
| DELETE returns 404 (note already gone) | Card removed anyway — desired state achieved | No error shown (silent success) |

---

## Design Rationale

- **Single confirmation step** — one `confirm()` call; not a multi-step modal (JRN-01.3: "not disruptive")
- **No route change** — confirmation happens on the same page (CP-03: No Navigation Required)
- **Optimistic removal** is optional in v1 — implementation may hide the card before API response returns, then restore on error
- **v1.1 opportunity:** Replace `window.confirm()` with an inline confirmation widget (e.g. "Are you sure? [Yes, delete] [Keep]" rendered within or below the card) for a less jarring UX
