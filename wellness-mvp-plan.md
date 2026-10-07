# Wellness MVP — Single-File `index.html` Plan

## Top-Level Overview

Build a single, self-contained `index.html` file — no libraries, no backend, no network calls, no external assets.  
Wellness professionals (or workshop attendees) can log mood, sleep, and water intake per day, view recent entries, see a 7-entry summary, and edit or delete any record.  
All data persists in `localStorage`. Every visible surface carries the label **"Demo only — use fictional data."**

---

## Data Shape

Each saved entry is a plain JSON object:

```
{
  "id":          string,   // crypto.randomUUID() or Date.now().toString()
  "date":        string,   // "YYYY-MM-DD" — user-supplied, must be unique per day
  "mood":        number,   // integer 1–5
  "sleepHours":  number,   // float 0–24, one decimal place accepted
  "waterGlasses":number,   // integer 0–20
  "notes":       string    // optional free text, max 200 chars, may be empty
}
```

Storage key: `"wellnessMvpEntries"` → JSON array of entry objects, sorted descending by date on read.

---

## Validation Rules

| Field | Rule |
|---|---|
| `date` | Required. Must match `YYYY-MM-DD`. Must not duplicate an existing entry's date (offer to edit instead). |
| `mood` | Required. Integer in [1, 5]. |
| `sleepHours` | Required. Number in [0, 24]. Accepts one decimal (e.g. 6.5). |
| `waterGlasses` | Required. Integer in [0, 20]. |
| `notes` | Optional. String ≤ 200 characters. Stripped of leading/trailing whitespace. |

On validation failure: inline error message beneath the offending field; do not clear other fields.

---

## Five-Step Test Checklist

1. **Add entry** — Fill all fields with valid data for today's date, click Save. Confirm the entry appears at the top of the Recent Entries list and in the 7-Entry Summary.
2. **Duplicate date guard** — Submit a second entry with the same date. Confirm an inline error appears and no duplicate is saved.
3. **Edit entry** — Click Edit on an existing entry, change the mood value, click Save. Confirm the list and summary reflect the updated value without adding a new row.
4. **Delete entry** — Click Delete on any entry, confirm the deletion prompt (or direct delete), and verify the entry is gone from both the list and the summary.
5. **Persistence** — After saving at least one entry, close and reopen the browser tab. Confirm all entries reload from `localStorage` exactly as saved.

---

## Sub-Tasks

---

### Sub-Task 1 — HTML Skeleton & Page Chrome

**Intent**  
Establish the single-file structure: `<!DOCTYPE html>`, semantic landmarks (`<header>`, `<main>`, `<section>`), the demo disclaimer banner, and placeholder sections for form, list, and summary. No logic yet.

**Expected Outcomes**
- File renders in a browser without errors.
- A clearly visible "Demo only — use fictional data." banner appears at the top.
- Three labeled sections are present: Entry Form, Recent Entries, 7-Entry Summary.
- Page title in `<title>` and `<h1>` reads "Wellness Log — Demo".

**Todo List**
- [ ] Create `index.html` with valid HTML5 boilerplate.
- [ ] Add `<header>` with app title and persistent demo disclaimer.
- [ ] Add `<main>` with three `<section>` elements: `#section-form`, `#section-list`, `#section-summary`.
- [ ] Add a `<footer>` repeating the demo disclaimer.

**Relevant Context**  
No existing files. Greenfield.

**Status** `[ ] pending`

---

### Sub-Task 2 — Embedded CSS

**Intent**  
Style the entire page inside a single `<style>` block. Readable, accessible, mobile-friendly — no framework.

**Expected Outcomes**
- Clean, readable layout on a 375 px mobile viewport and a 1024 px desktop.
- Form fields, buttons, list rows, and summary grid are visually distinct.
- Disclaimer banner is visually prominent (e.g. amber/yellow background, bold text).
- Focus indicators are visible (accessibility baseline).
- Error messages are styled in red beneath their fields.

**Todo List**
- [ ] Write CSS reset / box-sizing baseline inside `<style>`.
- [ ] Style the disclaimer banner (color, padding, text weight).
- [ ] Style the form: label/input stacking, consistent spacing, button primary style.
- [ ] Style inline validation error paragraphs (`.field-error`).
- [ ] Style the Recent Entries list (card or table row per entry, edit/delete button pair).
- [ ] Style the 7-Entry Summary (7 columns or a simple responsive grid showing the last 7 entries as mini-cards).
- [ ] Ensure `:focus-visible` outlines are not suppressed.

**Relevant Context**  
Sub-Task 1 must be complete (HTML landmarks must exist before styling them).

**Status** `[ ] pending`

---

### Sub-Task 3 — Data Layer (localStorage read/write)

**Intent**  
Implement all CRUD operations against `localStorage` as pure functions with no DOM dependencies. This isolates data logic for easy testing and keeps the event-handler code thin.

**Expected Outcomes**
- `loadEntries()` returns a parsed, date-descending sorted array (empty array if nothing stored).
- `saveEntries(arr)` serialises and writes the array back.
- `addEntry(entry)` validates, appends, saves, returns `{ok, error}`.
- `updateEntry(id, patch)` finds by id, merges patch, saves, returns `{ok, error}`.
- `deleteEntry(id)` removes by id, saves, returns `{ok}`.
- Duplicate-date check lives in `addEntry`.

**Todo List**
- [ ] Write `loadEntries()` / `saveEntries(arr)` inside a `<script>` block.
- [ ] Write `validateFields(fields)` returning an array of `{field, message}` error objects.
- [ ] Write `addEntry(fields)` using `validateFields` + duplicate-date check.
- [ ] Write `updateEntry(id, patch)` using `validateFields`.
- [ ] Write `deleteEntry(id)`.

**Relevant Context**  
Data shape and validation rules are defined in this plan's header sections.

**Status** `[ ] pending`

---

### Sub-Task 4 — Entry Form (add & edit mode)

**Intent**  
Wire the HTML form to the data layer. The same form handles both new entries and editing an existing entry (edit mode pre-populates fields and changes the submit button label).

**Expected Outcomes**
- Submitting a valid new entry calls `addEntry`, clears the form, and refreshes the list and summary.
- Inline errors appear beneath the correct field on validation failure; other fields retain their values.
- Clicking "Edit" on a list entry sets the form into edit mode (pre-filled, button reads "Update Entry").
- Submitting in edit mode calls `updateEntry`, exits edit mode, and refreshes list and summary.
- A "Cancel Edit" button exits edit mode and resets the form without saving.

**Todo List**
- [ ] Add form fields to `#section-form`: date, mood (select 1–5 with emoji labels), sleep hours (number), water glasses (number), notes (textarea), submit button.
- [ ] Write `renderFormErrors(errors)` to inject `.field-error` elements.
- [ ] Write `handleFormSubmit(e)` branching on edit mode flag.
- [ ] Write `enterEditMode(entry)` to pre-fill form and show "Cancel Edit".
- [ ] Write `exitEditMode()` to reset form and hide "Cancel Edit".

**Relevant Context**  
Depends on Sub-Task 3 (data layer functions must exist). Sub-Task 2 covers `.field-error` styling.

**Status** `[ ] pending`

---

### Sub-Task 5 — Recent Entries List & Delete

**Intent**  
Render all stored entries in reverse-chronological order. Each row has an Edit button (delegates to Sub-Task 4's edit mode) and a Delete button with a simple inline confirm.

**Expected Outcomes**
- `#section-list` renders one row per entry: date, mood (number or emoji), sleep, water, truncated notes.
- Delete triggers `window.confirm("Delete this entry?")` then calls `deleteEntry` and re-renders.
- Edit triggers `enterEditMode(entry)` and scrolls/focuses the form.
- Empty state shows a friendly placeholder message.
- List re-renders automatically after every add, update, or delete.

**Todo List**
- [ ] Write `renderList()` that reads `loadEntries()` and builds the list HTML via `innerHTML` or DOM API.
- [ ] Attach delegated click handler on `#section-list` for edit and delete actions using `data-id` attributes.
- [ ] Add empty-state message when no entries exist.
- [ ] Call `renderList()` on page load and after every mutation.

**Relevant Context**  
Depends on Sub-Tasks 3 and 4.

**Status** `[ ] pending`

---

### Sub-Task 6 — 7-Entry Summary

**Intent**  
Show the most recent 7 entries as a compact at-a-glance summary — no charts, no calculations beyond a simple visual grid of values. Clearly labelled as a summary view, not analysis.

**Expected Outcomes**
- `#section-summary` shows up to 7 most-recent entries as mini-cards (date + mood + sleep + water).
- If fewer than 7 entries exist, only those are shown.
- Summary re-renders after every mutation.
- Section heading reads "Last 7 Entries — Summary" (no trend language, no clinical language).

**Todo List**
- [ ] Write `renderSummary()` that slices the first 7 entries from `loadEntries()` and builds mini-cards.
- [ ] Display mood as both the number and a matching emoji for quick scan.
- [ ] Call `renderSummary()` on page load and after every mutation.

**Relevant Context**  
Depends on Sub-Task 3. Styling defined in Sub-Task 2.

**Status** `[ ] pending`

---

### Sub-Task 7 — Seed Data & Final Polish

**Intent**  
Pre-load three synthetic test entries so the page is not empty on first open. Add final demo-label pass and run the five-step test checklist manually.

**Expected Outcomes**
- On first load (empty `localStorage`), three fictional entries are pre-seeded matching the test personas from the product brief.
- All "Demo only — use fictional data." labels are present in header, footer, and summary section heading.
- Five-step test checklist passes completely.
- No `console.error` on load or during any CRUD action.

**Seed Entries (fictional)**

| Date | Mood | Sleep | Water | Notes |
|---|---|---|---|---|
| 2025-07-07 | 3 | 6.5 | 5 | "Busy Monday, skipped afternoon break" |
| 2025-07-08 | 2 | 5.0 | 3 | "Late night, felt sluggish all day" |
| 2025-07-09 | 4 | 7.5 | 8 | "Morning walk helped, felt more focused" |

**Todo List**
- [ ] Write `seedDemoData()` that checks if `localStorage` is empty before inserting seed entries.
- [ ] Call `seedDemoData()` before initial render on page load.
- [ ] Audit every user-visible string for any clinical, diagnostic, or prescriptive language — remove or neutralise.
- [ ] Manually run the five-step test checklist and confirm all pass.

**Relevant Context**  
Must run after all other sub-tasks are complete.

**Status** `[ ] pending`
