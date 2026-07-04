# Flow-03: Delete a Note with Confirmation

**User Story:** US-5.4, US-4.2, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.3 — Delete stage  
**Trigger:** User clicks (or keyboard-activates) the "Delete" button on a note card  
**Exit (confirmed):** Note card removed from list; empty state appears if last note  
**Exit (cancelled):** Card unchanged in read view  

---

## Flow Diagram

```
[Note card in READ VIEW]
[Shows: title, body, timestamp, "Edit" button, "Delete" button]
         │
         ▼
[User clicks/activates "Delete"]
         │
         ▼
[Confirmation prompt shown]
"Are you sure you want to delete this note?"
[OK / Cancel]
         │
         ├── User clicks CANCEL ───▶ [No API call]
         │                            [Card unchanged in READ VIEW]
         │                            [EXIT — Cancelled]
         │
         └── User clicks OK ────────▶ [Optional: card optimistically hidden]
                                                │
                                                ▼
                                   [DELETE /api/notes/:id]
                                                │
                                   ┌────────────┴─────────────┐
                                   │                          │
                              HTTP 204                  Non-204 / network
                                   │                          │
                                   ▼                          ▼
                          [Card permanently         [Restore card if
                           removed from list]        optimistically hidden]
                                   │                          │
                                   ▼                          ▼
                          [If last note:             Inline / toast error:
                           empty state appears]   "Failed to delete note.
                                   │               Please try again."
                                   ▼
                               [EXIT — Success]
```

---

## Steps Detail

### Step 1: User Clicks Delete
- "Delete" button is visible on note card in read view
- Reachable via Tab key; activatable via Enter or Space (US-5.5)
- Button is always visible (not hover-only — JRN-01.3 Select stage pain point)

### Step 2: Confirmation Prompt
- A confirmation prompt appears: **"Are you sure you want to delete this note?"** (US-5.4)
- Implementation: `window.confirm()` is acceptable for v1 (FRD F05)
- The prompt is synchronous — it blocks interaction until dismissed
- **If user cancels:** No API call is made; card remains in read view (US-5.4)
- **If user confirms:** Proceed to Step 3

### Step 3: Delete Request
- Optionally, the card is **optimistically hidden** from the list immediately after confirmation, before the API responds — for perceived responsiveness (FRD F05)
- Client sends: `DELETE /api/notes/:id`

### Step 4a: Success (HTTP 204)
- Card is permanently removed from the list (US-5.4)
- If the deleted note was the last one, the empty state message appears: **"No notes yet — add your first one above."** (US-4.2, US-5.4)
- No page reload required (US-4.3)

### Step 4b: Error (non-204 or network failure)
- If card was optimistically hidden, it is **restored** to the list (US-5.4)
- Inline or toast error message: **"Failed to delete note. Please try again."** (US-5.4)
- Card remains in read view; user may retry

### Edge Case: 404 on Delete
- If `DELETE` returns 404 (note already gone from DB), card is removed from list anyway
- Rationale: The desired state (note not in list) is already achieved (FRD F05 error table)

---

## Confirmation Prompt Design Note

The confirmation is a single-step prompt (`window.confirm`). It is:
- **Present but not disruptive** — does not block the full page with a custom modal (JRN-01.3 Delete stage)
- **Not a two-page flow** — no route navigation
- A custom inline confirmation widget is acceptable as a v1.1 enhancement if `window.confirm` is considered too jarring

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Confirmed delete | DELETE | `/api/notes/:id` | 204 No Content | Restore card; show error message |
