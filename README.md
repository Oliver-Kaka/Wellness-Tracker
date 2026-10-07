# Wellness Log — Demo Only

> **⚠ Demo only — fictional data only.**
> This is a learning prototype for wellness professionals. It is not a clinical tool, does not offer medical advice, and must not be used to store real personal or client health information.

---

## Table of contents

1. [Project overview](#1-project-overview)
2. [Architecture summary](#2-architecture-summary)
3. [Opening and previewing the app](#3-opening-and-previewing-the-app)
4. [Demo data: storage and erasure](#4-demo-data-storage-and-erasure)
5. [Privacy and data limitations](#5-privacy-and-data-limitations)
6. [Maintenance checklist](#6-maintenance-checklist)

---

## 1. Project overview

Wellness Log is a **single-file, zero-dependency HTML prototype** built for wellness professionals who want to explore what a simple self-tracking loop looks and feels like before committing to a full product.

A user logs three measurements per day:

| Field | Type | Range |
|---|---|---|
| **Mood** | Integer | 1 (Poor) – 5 (Great) |
| **Sleep hours** | Decimal | 0.0 – 24.0 hrs |
| **Water glasses** | Integer | 0 – 20 glasses |

Each entry also carries a **date** (one entry per day, duplicates are blocked) and an optional **notes** field (max 200 characters).

The app provides:

- A dated entry form with inline validation and edit mode
- A reverse-chronological entry list with per-entry edit and delete
- An at-a-glance **seven-entry summary** showing mini-cards and averaged statistics (entry count, average mood, average sleep, average water)
- Persistent storage via browser `localStorage`
- An "Erase all demo entries" control
- A built-in privacy and storage notice explaining the limitations of `localStorage`

**What this prototype is not:**

- Not a clinical tool
- Not a medical device or health record system
- Not suitable for storing real patient, client, or personal health data
- Not connected to any server, account system, or external service

---

## 2. Architecture summary

The entire application lives in one file: `index.html`.

```
index.html
├── <head>
│   ├── Embedded <style>          — all CSS, no external stylesheets
│   └── <meta robots: noindex>    — prevents accidental search-engine indexing
│
├── <body>
│   ├── .disclaimer banner        — "DEMO ONLY" notice, visible at all times
│   ├── <header>                  — app title
│   ├── <main>
│   │   ├── #section-form         — dated entry form (add + edit mode)
│   │   ├── <details.privacy-notice> — localStorage limitations disclosure
│   │   ├── #section-list         — recent entries list + "Erase all" control
│   │   └── #section-summary      — last-7-entries stats bar + mini-card grid
│   └── <footer>                  — repeated demo disclaimer
│
└── <script>                      — all JavaScript, no external libraries
    ├── Data layer
    │   ├── loadEntries()         — reads + sorts from localStorage
    │   ├── saveEntries(arr)      — serialises array to localStorage
    │   ├── validateFields()      — returns [{field, message}] error list
    │   ├── addEntry()            — validates, checks for duplicate date, saves
    │   ├── updateEntry()         — validates, merges patch, saves
    │   └── deleteEntry()         — filters by id, saves
    ├── Key migration
    │   └── migrateOldKey()       — one-time move from legacy key on startup
    ├── Seed data
    │   └── seedDemoData()        — three fictional entries on first open
    ├── DOM helpers
    │   └── makeEl(tag, props)    — createElement wrapper; all user text via
    │                               .textContent — never innerHTML
    ├── Renderers
    │   ├── renderList()          — entry cards with Edit / Delete buttons
    │   ├── renderStats()         — 4 stat tiles: count, avg mood, sleep, water
    │   └── renderSummary()       — 7 mini-cards showing per-entry values
    └── refreshAll()              — calls renderList + renderStats + renderSummary
```

### localStorage schema

All entries are stored as a JSON array under the key `wellness-tracker-demo-v1`.

Each entry object:

```json
{
  "id":           "uuid-or-timestamp-string",
  "date":         "YYYY-MM-DD",
  "mood":         3,
  "sleepHours":   7.5,
  "waterGlasses": 8,
  "notes":        "Optional free text, max 200 characters"
}
```

### Seven-entry summary

The stats bar and mini-card grid always operate on the **most recent seven entries** (sorted descending by date). Averages are simple arithmetic means, rounded to one decimal place. They are labelled as *"personal demo trends only — not medical findings"* in the UI.

---

## 3. Opening and previewing the app

### Prerequisites

- A modern browser (Chrome, Firefox, Safari, Edge — any current release)
- No Node.js, no build step, no server required

### Step 1 — Clone the repository

```bash
git clone https://github.com/Oliver-Kaka/Wellness-Tracker.git
cd Wellness-Tracker
```

### Step 2 — Open the file

**Option A — Double-click (simplest)**

Double-click `index.html` in your file manager. It opens directly in your default browser using a `file://` URL. All features including `localStorage` work normally.

**Option B — From the terminal**

```bash
# macOS
open index.html

# Windows (PowerShell)
Start-Process (Resolve-Path ./index.html).Path

# Linux
xdg-open index.html
```

**Option C — IBM Bob desktop environment**

1. Open the Explorer panel in Bob (folder icon, left sidebar).
2. Navigate to and click `index.html`.
3. Bob renders an inline HTML preview — no server or browser launch needed.

**Option D — VS Code Live Server** *(if the extension is installed)*

Right-click `index.html` in the VS Code Explorer → **Open with Live Server**. The page auto-refreshes on save.

### Static hosting — for demo sharing only

You may host this file on a static service (GitHub Pages, Netlify, etc.) **solely to share the fictional-data demo** with collaborators or workshop attendees. If you do:

- Remind all recipients that the app is **demo only — fictional data only**
- Each viewer's data is isolated to their own browser; no data is shared between users
- **Never use a hosted instance to collect real health information**
- The `<meta name="robots" content="noindex, nofollow">` tag is already present to discourage search-engine indexing

---

## 4. Demo data: storage and erasure

### How entries are stored

When you click **Save Entry**, the JavaScript:

1. Validates all fields (date format, mood 1–5, sleep 0–24, water 0–20, notes ≤200 chars)
2. Assigns a unique ID using `crypto.randomUUID()` (or `Date.now()` as fallback)
3. Appends the new entry to the existing array
4. Writes the full array back to `localStorage` under the key `wellness-tracker-demo-v1`

On page load the array is read back, sorted newest-first, and rendered. **No network request is made at any point.**

### Seed entries

On the very first load (empty `localStorage`), three fictional entries are pre-loaded automatically so the app is not blank:

| Date | Mood | Sleep | Water | Notes |
|---|---|---|---|---|
| 2025-07-07 | 3 / Okay | 6.5 h | 5 glasses | "Busy Monday, skipped afternoon break" |
| 2025-07-08 | 2 / Low | 5.0 h | 3 glasses | "Late night, felt sluggish all day" |
| 2025-07-09 | 4 / Good | 7.5 h | 8 glasses | "Morning walk helped, felt more focused" |

These are entirely fictional. They are not drawn from any real person.

### Erasing data

**Method 1 — In-app button**

Click **⚠ Erase all demo entries** (top-right of the Recent Entries section). A confirmation dialog appears. Confirming removes the `wellness-tracker-demo-v1` key from `localStorage` and clears the UI immediately.

**Method 2 — Browser developer tools**

1. Open DevTools (`F12` or `Cmd+Option+I`)
2. Go to **Application** (Chrome/Edge) or **Storage** (Firefox)
3. Expand **Local Storage** → select the origin
4. Delete the `wellness-tracker-demo-v1` key, or click **Clear All**

**Method 3 — Per-entry delete**

Each entry card has a **Delete** button. A confirmation dialog appears before removal.

---

## 5. Privacy and data limitations

> These limitations are also shown inside the app in the collapsible **🔒 Data storage & privacy** panel.

| Limitation | Detail |
|---|---|
| **Browser-local only** | `localStorage` stores data in the browser on the current device only. Data does not leave the device and is not transmitted anywhere. |
| **Not encrypted** | `localStorage` content is stored in plain text. Anyone with access to this browser profile — or to the device — can read, modify, or delete the data using browser developer tools. |
| **Not suitable for real data** | Do not enter real names, real dates of birth, real mood or health records, or any information that could identify a real person or client. |
| **No accounts or server** | There is no login, no database, no backend, and no cloud storage. The file makes zero outbound network requests. You can verify this in the browser's **Network** DevTools tab. |
| **Not a clinical tool** | Nothing in this prototype constitutes medical advice, a clinical assessment, a diagnosis, or treatment guidance of any kind. |
| **Scope of averages** | The seven-entry stats bar shows simple arithmetic averages of fictional logged values. They carry no clinical meaning and should not be interpreted as health trends. |

---

## 6. Maintenance checklist

Use this checklist whenever you change a field, add a new metric, adjust an input range, or refactor the JavaScript.

### A. Changing a field or input range

- [ ] **HTML form** — update the `<input>` or `<select>`: `min`, `max`, `step`, `placeholder`, and the `<label>` hint text
- [ ] **`validateFields()`** — update the guard condition and the error message string for that field
- [ ] **`addEntry()` / `updateEntry()`** — update the coercion (`parseInt`, `parseFloat`) and `toFixed()` precision if the type changes
- [ ] **`renderList()`** — update the pill text that displays the field value on each entry card
- [ ] **`renderStats()`** — if the field feeds the stats bar, update the tile label, getter, and unit string
- [ ] **`renderSummary()`** — update the mini-card row for that field
- [ ] **Seed data** — update `seedDemoData()` values to stay within the new valid range
- [ ] **`maxlength` attribute** — if notes limit changes, update both the HTML `maxlength` attribute and the JS validation threshold together

### B. Retesting after any change

Run each step manually in the browser. Check the browser console for errors (`F12 → Console`) throughout.

**1 — Save a valid entry**
- Fill all fields with values inside the new valid ranges
- Click **Save Entry**
- Confirm: entry appears at the top of Recent Entries
- Confirm: entry appears in the seven-entry summary grid
- Confirm: stats bar values update correctly

**2 — Reload persistence**
- Close the browser tab entirely
- Reopen `index.html`
- Confirm: all previously saved entries reload correctly
- Confirm: stats bar averages match the saved data

**3 — Average calculation**
- Log entries with known values (e.g., mood 2, 4, 4 → expected avg 3.3)
- Confirm the stats bar shows the correct rounded average
- Delete one entry and confirm the average updates immediately

**4 — Boundary / validation**
- Submit the form with each required field empty — confirm an inline error appears under the correct field only; other fields retain their values
- Submit with a value one step outside each boundary (e.g., mood = 0, sleep = 24.5) — confirm the error message names the correct field and valid range
- Submit a duplicate date — confirm the error message suggests using Edit

**5 — Edit flow**
- Click **Edit** on an existing entry
- Confirm the form pre-fills, the heading shows the blue edit banner, and the button reads **Update Entry**
- Change a value and click **Update Entry**
- Confirm the entry updates in the list and summary without creating a duplicate

**6 — Delete flow**
- Click **Delete** on any entry
- Confirm the browser confirmation dialog appears
- Confirm: after confirming, the entry is removed from the list, the summary grid, and the stats bar
- Click **⚠ Erase all demo entries** and confirm all entries are removed and the empty-state message appears

**7 — Keyboard access**
- Tab through every form field in order — confirm visible focus rings appear on each element
- Complete and submit the form using only the keyboard (Tab, Enter, arrow keys for the mood select)
- Tab to an **Edit** button and press Enter — confirm edit mode activates
- Tab to a **Delete** button and press Enter — confirm the confirmation dialog appears
- In edit mode, Tab to **Cancel Edit** and press Enter — confirm the form resets without saving

**8 — Narrow viewport (320 px)**
- Open browser DevTools → toggle device toolbar → set width to 320 px
- Confirm the form fields stack to a single column
- Confirm the stats bar collapses to a 2-column grid
- Confirm entry cards remain readable with no horizontal overflow
- Confirm all buttons are reachable and tappable

---

## Repo contents

```
.
├── index.html      — complete single-file application
└── README.md       — this file
```

---

> **Reminder:** This prototype is for learning and demonstration purposes only, using fictional data. It is not a medical device, does not store real health information, and does not offer clinical advice of any kind.
