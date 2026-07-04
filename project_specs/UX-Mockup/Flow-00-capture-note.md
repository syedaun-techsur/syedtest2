# Flow-00: Capture a Note

**User Story:** US-3.1, US-3.2, US-3.3, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.1 — Capturing a Thought Mid-Meeting  
**Trigger:** User opens or switches to the QuickNotes browser tab  
**Exit:** New note card appears at top of the list; form is cleared  
**Performance target:** POST + UI update < 500 ms (PRD §6)

---

## Flow Diagram

```
[User opens QuickNotes tab]
         │
         ▼
[Page loads — Compose Box visible above fold]
         │
         ▼
[User clicks into Body textarea (or auto-focused)]
         │
         ▼
[User types note body]
         │
         ├──── body.trim() === "" ──▶ [Add Note button stays DISABLED]
         │                                      │
         │                                      ▼
         │                             [User continues typing]
         │
         ├──── body.trim() ≠ "" ────▶ [Add Note button becomes ENABLED]
         │
         ▼
[User optionally types Title]
         │
         ▼
[User clicks "Add Note" (or keyboard submit)]
         │
         ▼
[Button → LOADING state ("Saving…"), temporarily disabled]
         │
         ▼
[POST /api/notes → { title: trimmed|null, body: trimmed }]
         │
         ├── HTTP 201 ──────────────▶ [Clear form fields]
         │                                      │
         │                                      ▼
         │                           [Re-fetch GET /api/notes]
         │                                      │
         │                                      ▼
         │                           [New note card appears at top of list]
         │                                      │
         │                                      ▼
         │                           [Button returns to DISABLED (body now empty)]
         │                                      │
         │                                      ▼
         │                                  [EXIT — Success]
         │
         └── Non-201 / network error ▶ [Show inline error near form]
                                                │
                                                ▼
                                   "Failed to save note. Please try again."
                                                │
                                                ▼
                                   [Form NOT cleared — values preserved]
                                                │
                                                ▼
                                   [Button returns to enabled/disabled per body]
                                                │
                                                ▼
                                          [User retries]
```

---

## Steps Detail

### Step 1: Page Load
- Compose box is rendered in SSR HTML (Next.js server-side render, US-3.4)
- Body textarea receives focus (auto-focus optional; Tab navigation required)
- "Add Note" button renders with `disabled` attribute (body is empty on load, US-3.2)

### Step 2: User Input
- User types in Body textarea (primary field)
- As soon as `body.trim() !== ""`, the `disabled` attribute is removed from the button
- Title input is visible above body but not required; user may skip it
- Whitespace-only body keeps button disabled (US-3.2)

### Step 3: Submit
- User clicks "Add Note" or activates via keyboard (Enter/Space, US-3.4)
- Button shows loading state ("Saving…") and is temporarily disabled to prevent double-submit
- Client sends: `POST /api/notes` with `{ title: titleValue.trim() || null, body: bodyValue.trim() }`

### Step 4a: Success (HTTP 201)
- Both title input and body textarea are cleared to `""`
- Button returns to disabled state (empty body)
- `GET /api/notes` is re-fetched
- New note card appears at top of the list without page reload (US-4.3)

### Step 4b: Error (non-201 or network failure)
- Inline error message appears near the compose box: **"Failed to save note. Please try again."** (US-3.3)
- Form values are preserved (not cleared) so user can retry
- Button returns to normal enabled/disabled state (not stuck in loading)

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Submit | POST | `/api/notes` | 201 + note JSON | Show inline error |
| After success | GET | `/api/notes` | 200 + note array | Retain old list; optional silent retry |
