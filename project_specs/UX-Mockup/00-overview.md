# UX Mockup — QuickNotes
# 00: Overview & Design Principles

**Project:** QuickNotes  
**Acronym:** quicknotes2  
**Generated:** 2026-07-04  
**Based on:** UserStories-quicknotes2.md, PRD-quicknotes2.md, FRD-quicknotes2.md, JOURNEYS-quicknotes2.md  

---

## UX Approach

QuickNotes is a **single-page, zero-navigation** note-taking application. Every interaction — composing, viewing, editing, deleting — happens on the single root page (`/`). There are no detail pages, no edit routes, no modals that block the full screen. This is a deliberate product constraint derived from persona research (JRN-01.1, JRN-01.3, CP-03): Alex Rivera cannot afford to navigate away from the main page during a meeting.

The UX is organised into two permanent vertical zones:

1. **Top zone — Compose Box (F3):** Always visible above the fold. Captures new notes.
2. **Bottom zone — Note List (F4/F5):** All persisted notes, newest first. Each card exposes inline edit and delete.

---

## Design Principles

### P1: Speed Over Polish
A note must be capturable in under 5 seconds from app open (PRD §7, US-3.1). Every UX decision that adds latency, friction, or navigation is rejected. The compose box body textarea is the primary field; title is secondary.

### P2: Immediate Feedback at Every State Change
Every action produces an unambiguous, immediate signal — button loading state, card update in place, empty-state appearance/disappearance. Silence after an action is a failure state (JRN cross-pattern CP-01).

### P3: Fail Loudly, Never Silently
Inline error messages appear near the action point when API calls fail. Forms are preserved (not cleared) on error. The user always has a clear path to retry (US-3.3, US-5.3, US-5.4).

### P4: No Navigation Required
All CRUD flows execute on the single root page. No `/notes/:id`, no `/notes/new`, no modals requiring route changes (CP-03).

### P5: Accessibility is Structural, Not Additive
Labels, keyboard navigation, ARIA roles, and `<time>` elements are required from initial implementation (US-3.4, US-5.5). Hover-only controls are explicitly rejected (JRN-01.3 Select stage).

---

## Page Structure (High-Level)

```
┌─────────────────────────────────────────────────────────┐
│  HEADER: "QuickNotes"  <h1>                             │
├─────────────────────────────────────────────────────────┤
│  COMPOSE BOX  (always visible, always above the fold)   │
│  ┌───────────────────────────────────────────────────┐  │
│  │  [Title input — optional]                         │  │
│  │  [Body textarea — required, primary focus]        │  │
│  │  [Add Note button — disabled when body empty]     │  │
│  │  [Inline error zone — appears on failure]         │  │
│  └───────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────┤
│  NOTE LIST  (scrollable, newest first)                  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Note Card (read view)                            │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  Title (or "Untitled") | Timestamp           │  │  │
│  │  │  Body text                                   │  │  │
│  │  │  [Edit] [Delete]                             │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  Note Card (edit mode)                            │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  [Title input — pre-filled]                  │  │  │
│  │  │  [Body textarea — pre-filled]                │  │  │
│  │  │  [Save] [Cancel]                             │  │  │
│  │  │  [Inline error zone — appears on failure]    │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  ...more cards...                                 │  │
│  └───────────────────────────────────────────────────┘  │
│  OR                                                     │
│  ┌───────────────────────────────────────────────────┐  │
│  │  "No notes yet — add your first one above."       │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## Persona Summary

| Persona | Primary UX Concern | Key Flows |
|---|---|---|
| PER-01 Alex Rivera (Knowledge Worker) | Speed of capture; trust that notes are saved | Flow-00 (Capture), Flow-01 (Review), Flow-02 (Edit), Flow-03 (Delete) |
| PER-02 Jordan Kim (Developer) | API contract correctness; fail-fast startup | Flow-04 (Health + API verification) — primarily non-UI |

---

## Story Coverage Map

| User Story | UX Section |
|---|---|
| US-3.1 Compose and Submit a New Note | Screen-00 Compose Box, Flow-00 |
| US-3.2 Prevent Empty Note Submission | Screen-00 Compose Box (disabled state) |
| US-3.3 Handle Compose Box Errors | Screen-00 Compose Box (error state) |
| US-3.4 Compose Box Accessibility | Y2-accessibility.md |
| US-4.1 View All Notes on Page Load | Screen-01 Note List, Flow-01 |
| US-4.2 View Empty State | Screen-01 Note List (empty state) |
| US-4.3 Note List Updates Without Reload | Flow-00, Flow-02, Flow-03 |
| US-5.1 Edit a Note Inline | Screen-02 Note Card Edit, Flow-02 |
| US-5.2 Cancel an Inline Edit | Screen-02 Note Card Edit (cancel) |
| US-5.3 Handle Inline Edit Errors | Screen-02 Note Card Edit (error state) |
| US-5.4 Delete a Note with Confirmation | Screen-03 Delete Confirmation, Flow-03 |
| US-5.5 Edit/Delete Keyboard Accessible | Y2-accessibility.md |
| US-0.1, US-0.2 | Flow-04 (developer, non-UI) |
| US-1.1–1.3 | Flow-04 (developer, non-UI) |
| US-2.1–2.5 | API layer, supports all UI flows |
