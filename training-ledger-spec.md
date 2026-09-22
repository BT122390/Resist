# Build Spec — Training Ledger v1 (block-card design)

Single-file, iPhone-first PWA for daily training. Reads the plan from Airtable, writes logs and day notes back. **The approved mockup is `blocks-mockup-v2.html` — match it for layout, type, color, spacing, and interaction.** This spec covers data flow, rules, and edge cases the mockup can't show. Supersedes `ledger-build-spec.md` and `bearlog-spec.md`.

## Repo & hosting
- New GitHub repo, GitHub Pages. Lives alongside `gymlog` and `fitlog`; neither is touched.
- `index.html` (React 18 + Babel standalone via CDN, no build), `manifest.json`, 180px icon, `README.md`, `UPDATES.md` (changelog, gymlog format).
- PWA: `apple-mobile-web-app-capable`, status bar `black-translucent`, safe-area insets, 44px min tap targets.

## Airtable
Base `appmqPkBzBzcKlrce` ("Training").

| Table | Fields used | Access |
|---|---|---|
| Exercises | `Name`, `Coaching Notes` | read-only |
| Schedule | `Day of Week`, `Order`, `Section`, `Block` (`fld2GSMfso7wBQZrt`), `Week Of` (`fld11dPwTJyXCHDXH`, ISO date), `Exercise` (link — **may hold more than one record**), `Prescription` | read-only |
| Logs | `Exercise` (link), `Log Date`, `Log Text` | read/write |
| Races | `Name`, `Date`, `Status` | read-only |
| Day Notes (`tblxTF4kmjHnGv4fx`) | `Date` (primary, ISO date, `fldTYmmQOFCd4x8V4`), `Notes` (long text, `fldqHY40fQEWHG43i`) | read/write |
| Settings (`tbl6UNTYuMVQ1u4mj`) | legacy fitlog table — **ignore in v1** | — |

Table IDs: Exercises `tblQ39nQ8p8HRSsIC`, Schedule `tblZP0Fg1dKWJzII2`, Logs `tblvJWkX10l1Fs6iE`, Races `tblusBLP1tM1sR7DH`.

**Schema is already in place** — Block, Week Of, and Day Notes were created 2026-09-19. Block is populated on Monday (Main Lifts ×6, Accessory ×8) and Wednesday (Sled Superset, Block A, Block B); all other rows are blank. All 70 rows have `Week Of` blank (template).

Rules:
- Logs link exercises via the `Exercise` field only. Never free text.
- A Schedule row's `Exercise` link can contain multiple records (e.g. Saturday #1 links both "Trail Session — Pre-Ruck Aerobic Primer" and "Trail Run"). Render one line per linked exercise, each with its own checkbox and Log; the row's `Prescription` shows under each.
- App never writes Exercises, Schedule, or Races.
- One Day Notes record per date; notes stored one per line. Upsert by date.
- PAT entered on first launch, stored in `localStorage`, never in repo. Clear and re-prompt on 401/403.

### Schedule resolution (Week Of overlay)
For a given date, compute the Monday of that week. If any Schedule rows have `Week Of` = that Monday, use **only** those rows for the whole week (a custom week fully replaces the template). Otherwise use template rows (`Week Of` blank). Rows for a day are filtered by `Day of Week`, sorted by `Order`.

### Card grouping
Cards on a day are built from `Block` when present, else `Section`. Monday's Main Lifts card lists all six lifts (Week A and Week B pairs) — the A/B alternation is not modeled in Airtable and is not the app's job in v1. Card order follows the lowest `Order` within the card. A card's optional prose description = the `Prescription` of the first row in the card **when that row's Exercise name matches the Block name** (e.g. a "The Ruck" block whose first row is a description row); otherwise no description. Notes card is always last.

### Loading
- Launch: fetch all Exercises, Schedule, Races (paginate `offset`). Hold in memory. Refresh Schedule on app foreground if last fetch > 1 hr.
- Per date: Logs where `{Log Date} = 'YYYY-MM-DD'` + Day Notes for that date. Cache per date; prefetch the visible week's other six days in the background for the week-strip dots.
- Last result per exercise (for the line dropdown): `filterByFormula={Exercise}="name"`, sort `Log Date` desc, `maxRecords=2`, exclude the current date.
- Exercises tab history: `maxRecords=3`, "Show more" fetches 10.
- Journal: Logs sorted `Log Date` desc, `pageSize=100`, infinite scroll. Day Notes fetched per rendered date.
- PRs: Logs for the four benchmark exercises (configured as a constant list of Exercise names in `index.html`), sorted by date; parse the leading number/time from `Log Text` for the trend bars; show raw text if unparseable.
- Writes optimistic, one retry, then an inline red "Retry" on the affected line. Never block input.

## Screens

### Header (all tabs)
Dark block with rounded bottom corners. Row 1: race pill (next Races record with `Date >= today`, `Status ≠ Completed`; `Name · N days`; hidden if none) left; calendar icon (date picker) and theme toggle right. Title: small-caps weekday over the date (Week) or tab label. Week strip below on the Week tab only: seven days Mon–Sun, today in red, selected day with a dark pill, green dot under any day with ≥1 log. Tap a day to select; swipe the card area left/right to move ±1 week (strip updates).

### Week
- Cards per block. Card header = block name + `done / total`. Optional description paragraph. Lines = round checkbox · exercise name · chevron.
- Checkbox tap toggles completion without opening the line (`stopPropagation`). Check with no result → Log with `Log Text = "Completed"`. Uncheck → delete if text is still `Completed`, else confirm sheet.
- Chevron/line tap opens the prescription dropdown: prescription in bold, then `Last: date — text` ×2. One line open per card at a time.
- **Log Result** button opens the block sheet: one field per exercise in the block, prefilled with today's existing results, prescription in small text under each name, single **Save N results** button. Saves create/update Logs and check the lines. Empty fields are left alone (not saved, not checked).
- **History** button opens a sheet with the last 5 results for each exercise in the block.
- **Notes** card: green accent, list of the day's notes, **Add Note** opens a one-field sheet. Swipe-left on a note to delete.

### Journal
- Days newest first: small-caps date label, then one card per block that has ≥1 log, lines = exercise + result. Skipped entries (Log Text starts with `Skipped`) struck through. Off-schedule logs go in an "Unscheduled" card. Notes card if Day Notes exists.
- Tap a line → Week tab on that date with that line's dropdown open.

### Exercises
- Search field, one card of scheduled exercises alphabetical, second card "Not scheduled" (archived / no Schedule row) grayed.
- Line tap → prescription + day, coaching notes, last 3 results, Show more.

### PRs
- One card per benchmark: name, latest value large, delta vs previous in green/red, date + cadence note, 5-bar trend. Tap → full history sheet.

### Center Log button
Sheet with two actions: **Log an exercise** (searchable Exercise picker, result field, date defaulting to selected day) and **Add a day note**. Creates a Log or appends to Day Notes.

## Visual tokens (from mockup)
Light: header `#1C1C1E`, bg `#F2F2F6`, card `#FFFFFF`, text `#1C1C1E`, muted `#6E6E73`, faint `#C7C7CC`, red `#E5484D`, red-soft `#FDE8E8`, green `#2FA46A`, green-soft `#E3F6EA`, hairline `#ECECF0`.
Dark: header `#000000`, bg `#0F0F11`, card `#1C1C1E`, text `#F2F2F5`, muted `#9A9AA0`, faint `#4A4A50`, red `#F0575C`, red-soft `#3A1E20`, green `#4FC37F`, green-soft `#17302A`, hairline `#2A2A2E`.
Type: `-apple-system, "SF Pro Text", Helvetica Neue, sans-serif`. Card title 19px/700, lines 17px/500, prescriptions 16px muted, header date 22px/700, week-strip numbers 19px, labels 11–13px bold small-caps. Cards 18px radius, 12px gap, 14px page padding. Buttons 12px radius, 13px vertical padding. Checkbox 22px circle, green when done. Theme follows system on first launch, then persists user choice.

## Out of scope for v1
Editing Exercises/Schedule/Races in-app; iPad layout; offline queue beyond one retry; charts beyond the PR trend bars; export.
