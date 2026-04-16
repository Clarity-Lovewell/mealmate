# MealMate — Meal Journal · SPEC v1.1

> **This document is the single source of truth for the MealMate app.**
> Upload this file alongside `index.html` at the start of every Claude session.
> Claude will update this file directly — you do not need to edit it manually.

---

## Purpose

A shared meal journal for two people — capturing what was eaten, building a living history of meals, and using that history to answer the daily question: *what are we having for dinner?*

The core difference from a recipe app or meal planner: MealMate learns from what you've actually eaten, not what you theoretically planned to cook. Meals are logged as they happen — with notes, photos, and ratings — and the AI uses that real history (plus your mood and energy tonight) to suggest genuinely fitting options rather than generic recommendations.

---

## Design Decisions

### Aesthetic

- Warm food-journal palette: cream background `#FDF6EC`, terracotta accent `#C4622D`, sage green secondary `#5C7A4E`, deep brown text `#2C1810`
- Fonts: **Playfair Display** (headings, meal names) + **DM Sans** (body, UI labels) — warm editorial serif paired with clean humanist sans
- Mobile-first: max-width 480px, bottom nav, large tap targets, camera-friendly photo capture
- Tactile card UI: white cards on cream, soft shadows, rounded corners
- No gamification, no streaks, no badges

### Navigation

- Bottom nav (3 items): **Tonight** · **Log** (centre, raised button) · **History**
- Settings opens as a bottom sheet via **gear icon top-right of Tonight screen**
- Meal detail opens as a bottom sheet modal from any meal card

### Tonight Screen

- Greeting with day and time period (e.g. "Monday evening")
- **Suggest dinner** hero card — tap to enter mood/preference flow
- Mood flow shown inline: Energy · Craving · Effort chips → "Get suggestions" → 3 AI suggestion cards
- "We're having this" button pre-fills the Log screen with that meal name
- **Recently eaten** section below — last 5 meals as compact cards

### Log Screen

- Form fields: meal name (text), category (chip select, dynamic), notes (textarea), voice dictation, photo, rating (1–5 stars), datetime
- Voice dictation appends to notes field via Web Speech API
- Photo stored as base64 in localStorage
- Saving navigates directly to History

### History Screen

- Horizontal filter strip: All + one chip per category (dynamic, matches current category list)
- Meals grouped by date with sticky date dividers
- Each meal card shows: photo, name, rating, notes snippet, time, category badge
- Category badges use inline colour styles (stable hash-based colour per category name)

### Meal Detail (Bottom Sheet)

- Full-width photo at top (if present)
- Meal name, category + rating + date/time as meta line
- Notes in a readable block

### AI Suggestions

- Requires Anthropic API key (stored in `mealmate_apikey`)
- Prompt includes: last 30 meals + mood chips + current category list
- Returns 3 suggestions as JSON with `name` and `reason`
- Model: `claude-sonnet-4-20250514`

### Voice Capture

- Web Speech API, no key required, `en-AU`, appends to notes field
- `continuous: true`, falls back to toast if unsupported

### Categories

- **Editable in v1.1** — add, rename (emoji + name), delete, or reset to defaults
- Stored in `localStorage` under `mealmate_categories` as `[{name, emoji}]`
- Default 10 categories (see table below)
- Category badge colours: stable hash-based assignment from a 10-colour palette (not CSS classes)
- All category UI (Log chips, History filter chips, History badges, AI prompt) renders dynamically from the stored list

| Category  | Default Emoji |
|-----------|---------------|
| Asian     | 🍜 |
| Italian   | 🍝 |
| Healthy   | 🥗 |
| Comfort   | 🫂 |
| Quick     | ⚡ |
| Takeaway  | 🥡 |
| Homestyle | 🥘 |
| Mexican   | 🌮 |
| Seafood   | 🐟 |
| Other     | ✨ |

---

## Data Structure

All data stored in `localStorage`:

| Key | Contents |
|-----|----------|
| `mealmate_v1` | `[{id, name, category, notes, rating, photo, date, createdAt}]` |
| `mealmate_apikey` | Anthropic API key string |
| `mealmate_categories` | `[{name, emoji}]` — falls back to DEFAULT_CATEGORIES if absent |
| `mealmate_devnotes` | `{active: [{id, text}], archived: [{id, text, archivedAt}]}` |

### Export / Import format (JSON)
```json
{
  "version": "1.0",
  "exportDate": "ISO timestamp",
  "meals": [...],
  "categories": [...],
  "devnotes": {...},
  "apikey": "sk-ant-..."
}
```

---

## Screens

| Screen | How to reach | Description |
|--------|--------------|-------------|
| Tonight | Nav: Tonight | AI suggestion flow, mood picker, recent meals |
| Log | Nav: Log (centre button) | Add a new meal |
| History | Nav: History | Full meal log, filterable, grouped by date |
| Meal Detail | Tap any meal card | Photo, name, meta, notes — bottom sheet |
| Settings | Gear icon on Tonight | All configuration — see Settings section |

---

## Settings (Bottom Sheet)

Organised into five sections:

### 📱 Install App
- Shows native install prompt if `beforeinstallprompt` fired (Android/Chrome with sw.js registered)
- Shows iOS-specific Add to Home Screen instructions if on iPhone/iPad
- Shows "Already installed" card if running in standalone mode
- Falls back to generic browser instructions otherwise

### 🔑 API Key
- Password input for Anthropic key — stored in `mealmate_apikey`
- "Save API Key" button — does not close the modal

### 📝 Dev Notes
- Text input + Add button (also adds on Enter key)
- Active notes list — each with checkbox and delete (×) button
- **Ticking a checkbox archives the note** (moves to Archived section after 300ms delay)
- Archived section: collapsible toggle, shows struck-through notes with delete option
- **Export notes as text** — downloads `.txt` file formatted for pasting into Claude:
  - "TO DO" section: numbered active notes
  - "DONE / ARCHIVED" section: ticked notes with ✓
  - Footer note to paste into Claude session

### 🏷️ Categories
- List of all current categories with emoji badge, name, edit (✏️), and delete (🗑️) buttons
- **Edit**: inline — replaces row with emoji input + name input + save/cancel buttons
- **Delete**: confirmation dialog; existing meals with that category name are not affected
- **Add new**: emoji input + name input + Add button (also adds on Enter key)
- **Reset to defaults**: confirmation dialog; restores the 10 default categories

### 💾 Data Backup
- **Export backup**: downloads `mealmate-backup-YYYY-MM-DD.json` with all data + API key
- **Import backup**: file picker for `.json` — shows confirmation with meal count before restoring

### ⚠️ Danger Zone
- **Clear all meal data**: removes `mealmate_v1` only (keeps categories, dev notes, API key)

---

## PWA / Install

### What's in the HTML
- PWA meta tags: `apple-mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style`, `apple-mobile-web-app-title`, `theme-color`
- Web app manifest injected via blob URL in JS (includes SVG icon, `display: standalone`, brand colours)
- `beforeinstallprompt` event listener — defers and stores the prompt for the Install button
- `appinstalled` event listener — updates install UI on success
- Attempts to register `./sw.js` — silently fails if file doesn't exist

### For Full Native Install on Android Chrome (optional)
Add a minimal `sw.js` to your GitHub repo root alongside `index.html`:

```javascript
self.addEventListener('install', () => self.skipWaiting());
self.addEventListener('activate', () => clients.claim());
self.addEventListener('fetch', e => {
  e.respondWith(fetch(e.request).catch(() => new Response('')));
});
```

With `sw.js` present, Chrome will fire `beforeinstallprompt` and the native "Install MealMate" button will appear in Settings.

### iOS
"Add to Home Screen" works out of the box in Safari — Settings shows step-by-step instructions.

---

## Key Flows

### Log a Meal
1. Tap **+** → Log screen → enter name (required), category, notes, voice, photo, rating, datetime
2. Tap **Save meal** → navigates to History

### Get Tonight's Suggestion
1. Tap **Suggest dinner** → mood picker → **Get suggestions** → spinner → 3 AI cards
2. Tap **We're having this** → Log screen pre-filled with name

### Manage Categories
1. Open Settings → Categories section
2. Edit (pencil) or delete (bin) existing categories inline
3. Add new with emoji + name → Add button
4. Reset to defaults if needed

### Use Dev Notes
1. Open Settings → Dev Notes section
2. Type note → Add (or press Enter)
3. Tick checkbox when done → note moves to Archived
4. Tap "Export notes as text" → download `.txt` → paste into Claude session at start of next dev session

### Backup / Restore
1. Settings → Data Backup → Export backup → saves JSON to device
2. On new device or after clearing: Import backup → select JSON → confirm → all data restored

---

## PWA / Mobile

- `max-width: 480px`, centred on desktop
- `min-height: 100dvh` on body
- Bottom nav uses `padding-bottom: env(safe-area-inset-bottom, 8px)` for iPhone notch
- Screen content uses `padding-bottom: 80px`
- No manifest or service worker bundled in HTML — see PWA section above for `sw.js` setup

---

## Hosting

Single HTML file (`index.html`) on GitHub Pages.
Optional: add `sw.js` to the same repo for native Android install.

---

## Session Workflow

- Chris provides: `index.html` + `MEALMATE_SPEC.md` at the start of each session
- Dev Notes export (`.txt`) can also be provided to brief Claude on pending changes
- **For small changes (1–2 features):** single prompt
- **For larger batches (3+ changes):** split into two prompts — code-heavy first, polish second
- Claude delivers: updated `index.html` + updated `MEALMATE_SPEC.md`

---

## CHANGELOG

| Version | Date | Change |
|---------|------|--------|
| v1.0 | 2026-04-14 | Initial build: Tonight screen with AI suggestion flow; Log screen (name, category, notes, voice, photo, star rating, datetime); History screen (filter strip, date-grouped cards, detail sheet, delete); Settings (API key, clear data); 10 categories; toast notifications; mobile-first layout |
| v1.1 | 2026-04-17 | PWA install support (manifest, `beforeinstallprompt`, iOS guide, sw.js instructions); Dev Notes in Settings (add, tick-to-archive, archived section, export as .txt); Editable categories (add/edit/delete/reset, dynamic rendering everywhere, hash-based badge colours); Export/Import all data as JSON (includes meals, categories, dev notes, API key) |

---

## Known Limitations

- No cross-device sync (data stays on one browser/device — use Export/Import to transfer)
- Photos stored as base64 — large photos consume significant localStorage space
- No search across meal history
- Voice capture works in Chrome/Edge; limited in Safari; not supported in Firefox
- AI suggestions require Anthropic API key and active internet connection
- No offline fallback for AI suggestions (recent meals always visible)
- Native Android install prompt requires `sw.js` to be added separately to the GitHub repo

---

## Planned Features

- [ ] Search across meal names and notes
- [ ] Weekly stats view (most eaten categories, average rating, meals per week)
- [ ] "Cook again" button on meal cards — pre-fills Log with that meal's name and category
- [ ] Filter by rating in History
- [ ] Shared meal planning — suggest a week of dinners at once
- [ ] Export meal history as CSV
- [ ] PWA manifest + service worker bundled fully for offline install
- [ ] Ingredient / recipe notes field (separate from general notes)

---

*MealMate SPEC v1.1 — Built with Claude Sonnet, April 2026*
