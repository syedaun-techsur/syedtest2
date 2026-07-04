# Y0: Interaction Patterns

**Applies to:** All screens and flows  
**User Stories:** US-3.1–3.4, US-4.1–4.3, US-5.1–5.5

---

## Pattern 1: Disabled Button as Validation Gate

**When to use:** Any form action where a required field must have content before submission.  
**Used in:** Compose Box "Add Note" button (US-3.2), Note Card "Save" button in edit mode (US-5.1)

**Behaviour:**
- The `<button>` element carries the native HTML `disabled` attribute (not CSS-only)
- The disabled state is driven by `fieldValue.trim() === ""`
- Whitespace-only input keeps the button disabled
- As soon as the user types a non-whitespace character, the button becomes enabled
- The button returns to its correct enabled/disabled state after any error (not stuck in loading)

**Why native `disabled` matters:** CSS-only visual disabling does not prevent keyboard users or assistive technology from activating the button. Native `disabled` prevents all activation paths.

**Example DOM:**
```html
<button type="submit" disabled>Add Note</button>   <!-- body empty -->
<button type="submit">Add Note</button>             <!-- body has content -->
```

---

## Pattern 2: Inline Error Near Action Point

**When to use:** Any API call that can fail, where the user needs to know they should retry.  
**Used in:** Compose Box on POST failure (US-3.3), Note Card on PUT failure (US-5.3), Note List on GET failure (US-4.3), Delete on DELETE failure (US-5.4)

**Behaviour:**
- Error message appears **near the element that triggered the action** (below the button, within the card)
- Error messages are concise and actionable:
  - POST fail: "Failed to save note. Please try again."
  - PUT fail: "Failed to save. Please try again."
  - PUT 404: "Note not found. It may have been deleted."
  - DELETE fail: "Failed to delete note. Please try again."
  - GET fail: "Failed to load notes. Refresh to try again."
- The error zone uses `role="alert"` or `aria-live="polite"` to announce to screen readers
- The error disappears when the user successfully retries or navigates away
- **Form values are preserved** on error — the user should never need to retype their content

**Placement rule:** Error message appears between the last form field and the footer of the component (below the button row).

---

## Pattern 3: Loading State on Submit Actions

**When to use:** Any action that triggers an API call with latency.  
**Used in:** "Add Note" button during POST (Flow-00), "Save" button during PUT (Flow-02)

**Behaviour:**
- Button label changes to "Saving…" (or similar) while awaiting API response
- Button is temporarily `disabled` during the request to prevent double-submission
- Both Save and Cancel buttons are disabled during the Save request in edit mode (prevents cancelling a save mid-flight)
- On completion (success or error), button returns to appropriate state
- **Target:** POST + UI update should complete in < 500 ms on local network, making the loading state brief but present

---

## Pattern 4: In-Place Card Transition (Read ↔ Edit)

**When to use:** Switching a note card between read view and edit mode.  
**Used in:** Note Card edit mode entry/exit (US-5.1, US-5.2)

**Behaviour:**
- The card transitions **in place** within the list — it does not navigate to a new page or open a modal
- In read view: title text, body text, timestamp, "Edit" and "Delete" buttons
- In edit mode: title input, body textarea, "Save" and "Cancel" buttons (replace Edit/Delete)
- Timestamp is hidden during edit mode (not relevant while editing)
- **Only one card** can be in edit mode at a time — opening a new edit (if implemented) auto-cancels the previous
- On Cancel: card reverts to read view instantly, no animation required
- On Save success: card updates to show new values from API response, no full list re-fetch

---

## Pattern 5: Optimistic List Update

**When to use:** After a create, edit, or delete action that succeeds.  
**Used in:** After POST (new card appears, Flow-00), after DELETE (card removed, Flow-03)

**Behaviour:**
- **Create (POST):** After success, re-fetch `GET /api/notes` and re-render the list. The new card appears at the top. No page reload.
- **Edit (PUT):** Update the card content locally from the PUT response body — no full list re-fetch required.
- **Delete (DELETE):** Remove the card from local state immediately on 204. Optionally, hide it optimistically before the API responds and restore on error.
- **Empty state transitions:** When the first note is created, the empty state message disappears immediately. When the last note is deleted, the empty state message appears immediately. Both without page reload (US-4.2).

---

## Pattern 6: Preserve Form State on Error

**When to use:** Any submit action that results in an API error.  
**Used in:** Compose Box on POST failure (US-3.3), Note Card on PUT failure (US-5.3)

**Behaviour:**
- On API error or network failure, **do not clear the form**
- The user's typed content is preserved so they can retry without retyping
- The inline error message appears near the button
- The button returns to its correct enabled/disabled state (not stuck in loading)
- This pattern is intentional — losing user input on error is a high-frustration failure mode (JRN-01.1 Submit stage)

---

## Pattern 7: Confirmation Before Destructive Action

**When to use:** Actions that permanently delete data.  
**Used in:** Delete note (US-5.4, Flow-03)

**Behaviour:**
- User must explicitly confirm before delete executes
- v1: `window.confirm("Are you sure you want to delete this note?")`
- On Cancel: no API call; card unchanged
- On Confirm: DELETE request proceeds
- Single confirmation step — not a two-step wizard (JRN-01.3 Delete stage: "not disruptive")
- v1.1 opportunity: inline confirmation widget within the card

---

## Pattern 8: Timestamp as Navigation Aid

**When to use:** Every note card, always.  
**Used in:** All note cards in read view (US-4.1, JRN-01.2 Scan/Identify stages)

**Behaviour:**
- Every card shows a **relative timestamp** derived from `note.created_at`
- Format: "just now", "2 minutes ago", "1 hour ago", "yesterday", "3 days ago"
- Rendered using: `<time dateTime="{note.created_at}">{relativeString}</time>`
- The `dateTime` attribute contains the ISO 8601 UTC string for machine readability
- Timestamps are Alex's **primary navigation mechanism** when titles are "Untitled" (CP-04)
- Static render on fetch is acceptable for v1; live ticking (auto-update without refetch) is optional
