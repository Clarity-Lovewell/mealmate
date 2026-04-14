# MealMate — Meal Journal · SPEC v1.0

> **This document is the single source of truth for the MealMate app.**
> Upload this file alongside `index.html` (and optionally a JSON backup) at the start of every Claude session.
> Claude will update this file directly — you do not need to edit it manually.

---

## Purpose

A shared meal journal for two people — capturing what was eaten, building a living history of meals, and using that history to answer the daily question: *what are we having for dinner?*

The core difference from a recipe app or meal planner: MealMate learns from what you've actually eaten, not what you theoretically planned to cook. Meals are logged as they happen — with notes, photos, and ratings — and the AI uses that real history (plus your mood and energy tonight) to suggest genuinely fitting options rather than generic recommendations.

### Relationship to Covenant and Clarity

MealMate is a **companion app** to Covenant and Clarity, not an extension of either. All three share the same technical architecture (single HTML file, localStorage, GitHub Pages, Anthropic API) but serve distinct purposes:

- **Covenant** — interior/spiritual life. Candlelight warmth, devotional tone.
- **Clarity** — exterior/operational life. Clean daylight, practical editorial tone.
- **MealMate** — shared domestic life. Warm food-journal aesthetic, tactile and inviting.

They form one personal ecosystem, each opened in its own context. MealMate is the one you reach for at 5pm.

---

## Design Decisions

### Aesthetic

- Warm food-journal palette: cream background `#FDF6EC`, terracotta accent `#C4622D`, sage green secondary `#5C7A4E`, deep brown text `#2C1810`
- Fonts: **Playfair Display** (headings, meal names, screen titles) + **DM Sans** (body, UI labels, notes) — warm editorial serif paired with clean humanist sans
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
- Mood flow (shown inline after tapping Suggest):
  - **Energy level**: Tired · Normal · Energetic
  - **Craving**: Comfort · Light · Bold · Noodles · Hearty
  - **Effort**: Order in · Quick cook · Proper cook
  - Chips (multi-selectable per group); tap "Get suggestions" to call AI
- AI returns exactly 3 suggestions — each shown as a card with name and reason
- "We're having this" button on each suggestion → pre-fills the Log screen with that meal name
- "Try different suggestions" resets the flow
- **Recently eaten** section below — last 5 meals as compact cards, tap to view detail

### Log Screen

- Form fields: meal name (text), category (chip select), notes (textarea), voice dictation, photo (camera/library), rating (1–5 stars), date/time
- Voice dictation appends to the notes field via Web Speech API
- Photo stored as base64 in localStorage — shown as full-width preview when added
- Saving a meal navigates directly to History

### History Screen

- Horizontal filter strip: All + one chip per category — filters the meal list
- Meals grouped by date with sticky date dividers (Today / Yesterday / weekday + date)
- Each meal card shows: photo (if present), name, rating stars, notes snippet, time, category badge
- Tap any card to open detail sheet
- Delete from within the detail sheet or from the card's action row

### Meal Detail (Bottom Sheet)

- Full-width photo at top (if present)
- Meal name, category + rating + date/time as meta line
- Notes in a readable block
- Close button top-right

### AI Suggestions

- Requires Anthropic API key (stored in localStorage, entered in Settings)
- Prompt includes: last 30 meals (name, category, rating, time since eaten) + selected mood chips
- AI returns structured JSON: 3 suggestions, each with `name` and `reason`
- Reason explains *why this makes sense tonight* (variety, mood fit, time since last eaten)
- No suggestion is shown until AI responds — spinner shown during fetch
- Errors shown as toast; mood picker hidden, recent meals shown instead

### Voice Capture

- 🎙 microphone button below the notes textarea on the Log screen
- Uses Web Speech API — no API key required
- Tap to start: button pulses red, label changes to "Tap to stop"
- Live interim transcription shown below the button; final transcripts appended to notes field
- Tap again to stop — final text committed
- **Android Chrome fix**: `continuous: false` with auto-restart on `onend` to simulate continuous recording
- `no-speech` and `aborted` errors restart silently; genuine permission errors show a toast
- Language: `en-AU`
- Falls back to a toast message if SpeechRecognition not supported

### Categories

10 predefined meal categories (not editable in v1):

| Category | Emoji | CSS class |
|---|---|---|
| Asian | 🍜 | cat-asian |
| Italian | 🍝 | cat-italian |
| Healthy | 🥗 | cat-healthy |
| Comfort | 🫂 | cat-comfort |
| Quick | ⚡ | cat-quick |
| Takeaway | 🥡 | cat-takeaway |
| Homestyle | 🥘 | cat-homestyle |
| Mexican | 🌮 | cat-mexican |
| Seafood | 🐟 | cat-seafood |
| Other | ✨ | cat-other |

Each category has a distinct badge colour used in History cards and the detail sheet.

---

## Data Structure

All data stored in `localStorage` under key `mealmate_v1`. API key stored separately under `mealmate_apikey`.

```
meals: [{
  id,           // timestamp string
  name,         // string
  category,     // string (category name) or null
  notes,        // string or ''
  rating,       // 1–5 or null
  photo,        // base64 data URL or null
  date,         // ISO timestamp (can be backdated via datetime-local input)
  createdAt     // ISO timestamp (same as date on creation)
}]
```

No separate settings object in v1 — API key is the only persisted setting beyond meal data.

---

## Screens

| Screen | How to reach | Description |
|---|---|---|
| Tonight | Nav: Tonight | AI suggestion flow, mood picker, recent meals |
| Log | Nav: Log (centre button) | Add a new meal — name, category, notes, voice, photo, rating, date |
| History | Nav: History | Full meal log, filterable by category, grouped by date |
| Meal Detail | Tap any meal card | Photo, name, meta, notes — bottom sheet |
| Settings | Gear icon on Tonight | API key entry, clear all data |

---

## Key Flows

### Log a Meal

1. Tap **+** (centre nav button) → Log screen
2. Enter meal name (required)
3. Select a category chip (optional)
4. Add notes via typing or voice dictation (optional)
5. Add a photo via camera or library (optional)
6. Rate 1–5 stars (optional)
7. Adjust date/time if logging retrospectively (defaults to now)
8. Tap **Save meal** → navigates to History

### Get Tonight's Suggestion

1. Tap **Suggest dinner** hero card on Tonight screen
2. Mood picker slides in — select energy level, craving, effort
3. Tap **Get suggestions** → spinner shown
4. AI returns 3 suggestions as cards with reasons
5. Tap **We're having this** on chosen option → navigates to Log screen with name pre-filled
6. Complete and save the log entry as normal

### Log from Suggestion

1. AI suggests "Beef stir fry" → tap "We're having this"
2. Log screen opens with "Beef stir fry" pre-filled in name field
3. Add notes, photo, rating → Save

### View History

1. Tap **History** nav tab
2. Scroll through meals grouped by date
3. Tap a category chip to filter
4. Tap any meal card to open detail sheet
5. Tap **Delete** in card actions to remove (confirmation dialog)

---

## Settings

Accessed via gear icon on the Tonight screen header. Opens as a bottom sheet.

- **API Key**: password input for Anthropic key — stored in `mealmate_apikey` in localStorage. Instructions to get key from `console.anthropic.com`.
- **Clear all data**: danger zone button — confirmation before wiping `mealmate_v1`

---

## PWA / Mobile

- No manifest or service worker in v1 — can be added in a future version
- `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">`
- Bottom nav uses `padding-bottom: env(safe-area-inset-bottom, 8px)` for iPhone notch
- Screen content uses `padding-bottom: 80px`
- `min-height: 100dvh` on body
- Max width 480px, centred — looks correct on all phone sizes and desktop

---

## Hosting

Single HTML file (`index.html`) on GitHub Pages.
Repo: `https://github.com/[username]/mealmate`
Live URL: `https://[username].github.io/mealmate/`
Workflow: Edit → rename `mealmate.html` to `index.html` → commit to main → push → live ~60 seconds

Sits alongside Covenant and Clarity as a companion repo on the same GitHub account.

---

## Session Workflow

- Chris provides: `index.html` + `MEALMATE_SPEC.md` + optional JSON backup at the start of each session
- **For small changes (1–2 features):** single prompt is fine
- **For larger batches (3+ changes):** split into two prompts — first handles code-heavy changes (data structure, new screens, new modals), second handles lighter changes (copy, UI polish, confirmation dialogs)
- Changes requested in plain language; Claude handles the code
- Claude delivers: updated `index.html` + updated `MEALMATE_SPEC.md`
- Chris commits both files to GitHub

---

## CHANGELOG

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-04-14 | Initial build: Tonight screen with AI suggestion flow (mood/energy/effort chips, 3 AI suggestions with reasons); Log screen (name, category chips, notes, voice dictation, photo capture, star rating, datetime); History screen (category filter strip, date-grouped meal cards, detail bottom sheet, delete); Settings bottom sheet (API key, clear data); 10 meal categories; toast notifications; mobile-first layout |

---

## Known Limitations

- No cross-device sync (data stays on one browser/device)
- Photos stored as base64 — large photos will consume significant localStorage space
- No search across meal history
- Categories cannot be renamed or customised in v1
- Voice capture works in Chrome/Edge; limited in Safari; not supported in Firefox
- AI suggestions require Anthropic API key and active internet connection
- No offline fallback for suggestions (recent meals are always visible as a fallback)

---

## Planned Features

- [ ] Search across meal names and notes
- [ ] Weekly stats view (most eaten categories, average rating, meals per week)
- [ ] Shared meal planning — suggest a week of dinners at once
- [ ] Custom categories
- [ ] Export meal history as CSV or PDF
- [ ] PWA manifest + service worker for offline install
- [ ] "Cook again" button on meal cards — pre-fills Log with that meal's name and category
- [ ] Ingredient / recipe notes field (separate from general notes)
- [ ] Filter by rating in History

---

*MealMate SPEC v1.0 — Built with Claude Sonnet, April 2026*
