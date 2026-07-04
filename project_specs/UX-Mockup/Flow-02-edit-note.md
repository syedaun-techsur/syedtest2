# Flow-02: Edit a Note Inline

**User Story:** US-5.1, US-5.2, US-5.3, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.3 — Correcting and Retiring Notes After Action  
**Trigger:** User clicks (or keyboard-activates) the "Edit" button on a note card  
**Exit (success):** Card returns to read view showing updated content  
**Exit (cancel):** Card returns to read view with original content unchanged  
**Performance target:** Card update in place < 500 ms after Save (JRN-01.3)

---

## Flow Diagram

```
[Note card in READ VIEW]
[Shows: title, body, timestamp, "Edit" button, "Delete" button]
         │
         ▼
[User clicks/activates "Edit"]
         │
         ▼
[Card transitions to EDIT MODE]
[Title input — pre-filled with note.title (or "" if null)]
[Body textarea — pre-filled with note.body]
["Save" and "Cancel" replace "Edit" and "Delete"]
[All other cards remain in READ VIEW — only one edit at a time]
         │
         ▼
[User modifies title and/or body]
         │
         ├──── body.trim() === "" ──▶ ["Save" button stays DISABLED]
         │
         ├──── body.trim() ≠ "" ────▶ ["Save" button ENABLED]
         │
         ▼
[User clicks "Save"]                    OR    [User clicks "Cancel"]
         │                                              │
         ▼                                              ▼
[Save button → LOADING ("Saving…")]          [Card reverts to READ VIEW]
[Both buttons temporarily disabled]          [Original values restored]
         │                                   [No API call made]
         ▼                                   [EXIT — Cancel]
[PUT /api/notes/:id → { title, body }]
         │
         ├── HTTP 200 ──────────────▶ [Card exits EDIT MODE]
         │                            [Card displays updated title, body,
         │                             updated_at from API response]
         │                            [No full list re-fetch needed]
         │                                      │
         │                                      ▼
         │                                  [EXIT — Success]
         │
         ├── HTTP 404 ─────────────▶ [Card stays in EDIT MODE]
         │                            Inline error:
         │                            "Note not found. It may have been deleted."
         │                            ["Save" and "Cancel" return to normal]
         │
         └── HTTP 400/500 / network ▶ [Card stays in EDIT MODE]
                                       Inline error:
                                       "Failed to save. Please try again."
                                       ["Save" and "Cancel" return to normal]
                                       [User retries or cancels]
```

---

## Steps Detail

### Step 1: Enter Edit Mode
- User clicks "Edit" button (visible on card in read view, reachable via Tab key, US-5.5)
- Card transitions **in place** — the note card expands/transforms to show inputs, no page navigation
- Title `<input type="text">` pre-filled with `note.title` or `""` if null (US-5.1)
- Body `<textarea>` pre-filled with `note.body` (US-5.1)
- "Save" and "Cancel" buttons replace "Edit" and "Delete" (US-5.1)
- All other cards remain in read view — only one card can be in edit mode at a time (US-5.1)

### Step 2: User Edits
- User modifies title and/or body freely
- "Save" button is disabled when `body.trim() === ""` (native `disabled` attribute, US-5.1)
- User may Tab between title input → body textarea → Save → Cancel (US-5.5)

### Step 3a: Save
- User clicks "Save" or activates via keyboard
- "Save" button shows loading state; both Save and Cancel temporarily disabled (prevents double-submit)
- Client sends: `PUT /api/notes/:id` with `{ title: trimmedTitle || null, body: trimmedBody }`

### Step 4a: Save Success (HTTP 200)
- Card exits edit mode and returns to read view
- Card displays values from the API response (updated title, body, `updated_at`)
- No full `GET /api/notes` re-fetch required — card state updated locally from PUT response (US-5.1)

### Step 4b: Save Error — 404
- Card stays in edit mode (US-5.3)
- Inline error: **"Note not found. It may have been deleted."**
- Save and Cancel buttons return to normal state

### Step 4c: Save Error — 400/500/network
- Card stays in edit mode (US-5.3)
- Inline error: **"Failed to save. Please try again."**
- Save and Cancel buttons return to normal state
- User retries or cancels

### Step 3b: Cancel
- User clicks "Cancel" at any point during edit mode (US-5.2)
- Card reverts to read view immediately
- Original title and body displayed (exact original values, not trimmed/modified intermediates, US-5.2)
- No API call is made
- "Edit" and "Delete" buttons reappear (US-5.2)

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Save | PUT | `/api/notes/:id` | 200 + updated note JSON | Show inline error; stay in edit mode |
