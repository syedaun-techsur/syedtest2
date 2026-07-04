# Flow-01: Review Notes on Page Load

**User Story:** US-4.1, US-4.2, US-4.3  
**Persona:** PER-01 Alex Rivera  
**Journey:** JRN-01.2 — Returning to Review and Act on Captured Notes  
**Trigger:** User opens `http://localhost:3000` or switches back to the QuickNotes browser tab  
**Exit:** User has scanned the note list and identified the target note  
**Performance target:** Page initial load with notes list rendered < 2 s (PRD §6)

---

## Flow Diagram

```
[User navigates to http://localhost:3000]
         │
         ▼
[SSR: Page HTML delivered with Compose Box rendered]
         │
         ▼
[Client hydrates — React takes over]
         │
         ▼
[GET /api/notes called on mount]
         │
         ├── Fetching ──────────────▶ [Note list area shows loading indicator]
         │                            (e.g. "Loading notes…" or skeleton cards)
         │
         ├── HTTP 200 — empty [] ──▶ [Empty state message displayed]
         │                            "No notes yet — add your first one above."
         │                                      │
         │                                      ▼
         │                           [Compose box remains fully usable]
         │                                      │
         │                                      ▼
         │                           [User creates first note → Flow-00]
         │                           [Empty state disappears; card appears]
         │
         ├── HTTP 200 — with notes ▶ [Render note cards, newest first]
         │                                      │
         │                                      ▼
         │                           [Each card shows: title or "Untitled",
         │                            body text, relative timestamp]
         │                                      │
         │                                      ▼
         │                           [User scans list — identifies target note]
         │                                      │
         │                                      ▼
         │                                  [EXIT — Flow-02 (Edit)
         │                                   or Flow-03 (Delete)]
         │
         └── HTTP 500 / network ────▶ [Error message in note list area]
                                       "Failed to load notes. Refresh to try again."
                                                │
                                                ▼
                                   [Compose box still fully usable]
                                   [User may manually refresh page]
```

---

## Steps Detail

### Step 1: Server-Side Render
- Next.js delivers the full page HTML including the Compose Box (US-3.4 SSR requirement)
- The note list area is rendered with a loading placeholder or left empty pending client fetch

### Step 2: Client Mount — Fetch Notes
- On React hydration, `GET /api/notes` is called immediately
- A loading indicator is displayed in the list area during the fetch

### Step 3a: Empty State
- If `GET /api/notes` returns `[]`, display: **"No notes yet — add your first one above."** (US-4.2)
- Exact message match required per acceptance criteria
- Compose box remains fully usable
- When user creates the first note (Flow-00), the empty state disappears and the new card appears without page reload (US-4.2)
- If last note is deleted (Flow-03), empty state reappears without page reload (US-4.2)

### Step 3b: Notes List Rendered
- One note card per note, in the API-returned order (newest first — no client re-sorting, US-4.1)
- Each card displays:
  - **Title:** `note.title` if non-null and non-empty; otherwise **"Untitled"** (visually distinguished — italic or muted colour, US-4.1)
  - **Body:** Plain text, no Markdown rendering
  - **Timestamp:** Relative time string (e.g. "2 minutes ago", "yesterday") rendered in a `<time>` element with `dateTime={note.created_at}` attribute (US-4.1)
  - **Controls:** "Edit" button and "Delete" button (F5, US-5.5)

### Step 4: Error State
- If `GET /api/notes` returns 500 or fetch throws, display: **"Failed to load notes. Refresh to try again."** (US-4.3)
- Compose box is still functional
- Previous list state is retained on refresh failure (US-4.3)

---

## API Calls in This Flow

| Step | Method | Endpoint | Expected | On Error |
|---|---|---|---|---|
| Page load | GET | `/api/notes` | 200 + note array | Show error message in list area |

---

## Information Hierarchy on Notes List

| Priority | Content | Why |
|---|---|---|
| Primary | Newest note (index 0) | Most likely to be the note the user just created or needs |
| Secondary | Relative timestamp | Alex's primary navigation aid (CP-04 from JOURNEYS) |
| Secondary | Body text (first ~2 lines visible) | Identification when title is "Untitled" |
| Tertiary | Title (or "Untitled" fallback) | Secondary identifier |
| Tertiary | Edit / Delete controls | On-demand actions |
