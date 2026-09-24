# Build Spec — Training Ledger v2 (Gmail-style design)

**Supersedes `training-ledger-spec.md`.** Single-file, iPhone-first PWA. Reads the plan from Airtable, writes logs, day notes, and — new in v2 — per-week schedule edits. **The approved mockup is `gmail-mockup-v6.html`; match it for layout, type, color, spacing, and gestures.** This spec covers data flow, rules, and edge cases the mockup can't show.

## Repo & hosting
Unchanged from v1: repo `BT122390/Resist`, GitHub Pages, `index.html` (React 18 + Babel via CDN), `manifest.json`, 180px icon, `README.md`, `UPDATES.md`. PWA meta tags, safe-area insets, 44px min targets.

## Airtable
Base `appmqPkBzBzcKlrce`. Table IDs: Exercises `tblQ39nQ8p8HRSsIC`, Schedule `tblZP0Fg1dKWJzII2`, Logs `tblvJWkX10l1Fs6iE`, Races `tblusBLP1tM1sR7DH`, Day Notes `tblxTF4kmjHnGv4fx`. Settings `tbl6UNTYuMVQ1u4mj` is legacy — ignore.

| Table | Fields | Access |
|---|---|---|
| Exercises | `Name`, `Coaching Notes` | read; **create** (new exercise from Add sheet) |
| Schedule | `Day of Week`, `Order`, `Section`, `Block` (`fld2GSMfso7wBQZrt`), `Week Of` (`fld11dPwTJyXCHDXH`), `Exercise` (link, may be multi), `Prescription` | read; **create/update/delete only on rows where `Week Of` is set**; template rows editable only via explicit "Every <weekday>" scope |
| Logs | `Exercise` (link), `Log Date`, `Log Text` | read/write |
| Races | `Name`, `Date`, `Status` | read |
| Day Notes | `Date` (`fldTYmmQOFCd4x8V4`), `Notes` (`fldqHY40fQEWHG43i`) | read/write |

Schema is in place (Block populated Mon/Wed; all 70 rows are template with `Week Of` blank).

Rules:
- Logs link exercises through `Exercise` only. Never free text.
- A Schedule row's `Exercise` may hold multiple records → render one line per exercise.
- One Day Notes record per date, one note per line.
- PAT entered on first launch → `localStorage`; never in repo; clear on 401/403.

### Week resolution
Monday of the selected date = `weekStart`. If any Schedule rows have `Week Of = weekStart`, those rows are the week (overlay); else template rows (`Week Of` blank). Day = filter `Day of Week`, sort `Order`.

### Materializing an overlay (copy-on-write)
The first edit to any day in a template week copies **all** template rows for that week into new rows with `Week Of = weekStart` (same Day of Week, Order, Section, Block, Exercise, Prescription), then applies the edit to the copies. Done as one batched create (10 per request). Show a brief toast: "Customizing week of Sep 21." Templates are never modified by this path.

### Edits
| Gesture | Default scope | Airtable write |
|---|---|---|
| Swipe-left → Remove | This week | delete overlay row (materialize first if needed) |
| Add → pick exercise | This week | create overlay row: `Day of Week`, `Week Of`, `Order` = max+10, `Exercise` link; `Section` = "Main Session"; `Block` blank; `Prescription` = the exercise's template prescription if it has one, else blank |
| Add → Create "…" | — | create Exercises record (`Name`), then the Schedule row as above |
| Drag ⋮⋮ → reorder | This week only (no template option) | rewrite `Order` for the day's overlay rows as 10, 20, 30… in one batched update |
| Scope "Every <weekday>" (on Remove and Add sheets) | — | same operation on the **template** row(s) instead; no materialization |

Remove shows a 5-second **Undo** toast instead of confirming (Undo recreates the row with the same fields and Order; if the exercise had a log today the log is untouched either way). Reorder does not confirm. All edits optimistic; on failure revert the list and show a red toast with Retry.

## Loading
- Launch: fetch Exercises, Schedule (all rows, template + overlays), Races. Refresh on foreground if > 1 hr old.
- Per date: Logs (`{Log Date}='YYYY-MM-DD'`) + Day Notes; cache per date; prefetch the visible week for the green dots.
- Last result per exercise for the list's third line: batch — one query per visible exercise, sorted `Log Date` desc, `maxRecords=1` excluding today; cache per exercise for the session.
- Search: exercise-name results = Logs where linked exercise name contains the query (client-side filter on the full Logs fetch, paginated `pageSize=100`, sorted desc; fetch until 12 months back or 2,000 logs, whichever first). Note matches = Day Notes where `Notes` contains the query. **Do not match on `Log Text`.**
- Week view (Search → week chip): Logs + Day Notes for the seven days, grouped by date.

## Screens

### Home (day list)
- **Top pill bar**: 7 round day buttons Mon–Sun for the current week, today filled red, selected day (if not today) outlined red, green dot under days with ≥1 log. Tap selects. Swipe the list left/right moves ±1 week (pill dates update). Long-press today → jump back to current week.
- Under the pill: full date left, race countdown right (next Races with `Date ≥ today`, `Status ≠ Completed`; hidden if none).
- **Rows** (Gmail 3-line): colored avatar with block initial · bold exercise name · prescription · muted last result (`Last: …`) or `Today: …` once logged · right-side date of last log, or green "Done". Avatar color/letter from `Block`, else `Section` (Warm-Up gray W, Main Session green M, Finisher red F, Recovery teal R; Blocks use a fixed palette by name, assigned in order of first appearance and persisted in localStorage). **No section headers.**
- **Avatar tap** = toggle done (stopPropagation): creates `Completed` log / deletes it (confirm if text ≠ Completed).
- **Row tap** = open Exercise Page.
- **Swipe left** = Remove (see Edits). **Drag handle** ⋮⋮ = reorder. Handle is hidden while a row is swiped open.
- **Notes row** is the **first row of every day**, on the light `--bar` background so it reads as a header: pencil avatar, first two notes as the preview lines, readiness dot + word on the right (blank if unset). Tap → Notes page. Not draggable, removable, or checkable.

### Notes page
Slides in like the Exercise Page. Top: back, date, ⋮ (copy all notes). Body: title "Notes", Readiness chips, add field with round save (appends a line to Day Notes, timestamped in the UI only — the stored line is plain text), today's notes newest first with the time they were added shown from the record's modified time if available else blank, then **"Earlier this week"** — notes from the other days of the same week with the weekday as the left label (read-only here; tap jumps to that day). Swipe-left on a today note deletes it (Undo toast).


### Readiness (on the Notes page)
Top of the Notes page: three round chips Green / Yellow / Red with label "Readiness" and a meaning line (Green — execute as written · Yellow — reduce intensity · Red — replace the session). Tap sets; tap again clears. Stored as the first line of that date's Day Notes in the fixed form `Readiness: Green`; the app strips that line from the visible notes list and shows it as chip state. The Home Notes row shows the color as a small dot plus its word where the date would be. Search week view shows the readiness dot beside each day label.

### Result save — pain and RPE chips
On the Exercise Page under the result field, two optional chip groups: **Pain** 0 · 1 · 2 · 3 and **RPE** 6 · 7 · 8 · 9 · 10. Selected values are appended to `Log Text` as ` · pain 2 · RPE 8` on save. Parsing rule for display and search: trailing ` · pain N` and ` · RPE N` segments are split off and rendered as small chips on rows and history lines. A result containing `pain 2` or higher renders its chip in amber; `pain 3` in red. Search gets a fixed chip **Pain ≥ 2** that filters logs by this segment (the only case where search reads `Log Text`).

### Plan next week (Sunday planning surface)
From the Add sheet when the selected day is in the current week, or from a ⋮ on the week pill bar: **"Plan next week"**. Behavior: if next week has no overlay, copy this week's resolved rows (overlay if present, else template) into next week's overlay (`Week Of` = next Monday), then jump the pill bar to next week with Monday selected and show the toast "Week of Sep 28 ready to edit." If an overlay already exists, just jump there. Also available: **"Reset week to template"** on the same menu — deletes that week's overlay rows after an Undo toast (10 s). The Sunday coaching review writes to next week's overlay rows through Airtable; this action is what lets Bryan see and adjust that plan in the app.

### Exercise Page
Slides in from the right. Top: back, `n of N` position, ⋮ menu (Skip today → log `Skipped`, Move to another day → day picker, Remove from this week, Open in Exercises). Body: avatar + name + subline (block · weekday · pairing text if the prescription mentions "paired with"), result field with round save, quick chips (last result; last result with first `\d+ lb` incremented by 5 when present; Skipped), Pain and RPE chips (see above), Prescription, Coaching Notes, History (5 + Show all). Bottom: Previous / Next with neighbor names; horizontal swipe does the same. Saving a result checks the row and advances to Next; on the last exercise it returns to the list.

### Search
Search bar with back and clear. Week chips beneath (This week, then prior weeks by date range, scrolling back as far as logs exist). Empty query → week view for the selected chip (compact Journal: one row per day — headline exercise + number, one-line result summary, "+ others" line, skipped items struck, no-log days faint; tap → that date on Home). Non-empty query → exercise results (every log for matching exercise names, newest first; tap → Exercise Page for that date) then "Also matches" for Day Notes.

### Exercises
Search field, alphabetical list, tap → Exercise Page in library mode (no date, no result field; prescription per scheduled day, notes, full history). Archived (no Schedule row) listed below a faint divider.

### Races
List from Races sorted by Date: name, date, city/state, days-until, status chip. Read-only. The bottom-bar badge shows days to the next race.

### Theme
Bottom bar has five buttons: Exercises, Races, Search, Theme, Add. Theme toggles light/dark; follows system on first launch, then persists.

### Add
Sheet: title "Add to <weekday>, <date>", search field, results (avatar with home-day letter, name, home day · prescription), "Create '<query>'" row, scope toggle (This week only / Every <weekday>). Tap + adds and closes; Create adds an Exercises record first.

## Visual tokens (from mockup)
Light: bg `#FFFFFF`, bar `#EEF2F6`, text `#1F1F1F`, muted `#5F6368`, faint `#C4C7CC`, hair `#EEEEEE`, red `#D93025`, blue `#1A73E8`, green `#188038`, red-soft `#FCE8E6`, blue-soft `#E8F0FE`, green-soft `#E6F4EA`. Dark: bg `#121212`, bar `#1F1F1F`, text `#E8EAED`, muted `#9AA0A6`, faint `#5F6368`, hair `#2A2A2A`, red `#F28B82`, blue `#8AB4F8`, green `#81C995`. Avatar palette: `#7B61FF`, `#E8710A`, `#1A73E8`, `#0B8043`, `#C5221F`, `#5F6368`. Type: Google Sans → -apple-system → Roboto. Row name 17/600, prescription 15, last result 14 muted, day buttons 42px, bottom buttons (5) 44px circles with 10px labels, remove swipe reveal 96px red.

## Out of scope for v2
Editing prescriptions or coaching notes in-app; multi-select; benchmark/PR tracking (dropped by design); iPad layout; offline queue.
