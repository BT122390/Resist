# Build Spec — Resist v3 (Clock-style design)

**Supersedes every earlier spec.** Single-file, iPhone-first PWA modeled on Apple's Clock app. Reads the plan from Airtable, writes logs, day notes, per-week schedule edits, new exercises, and races. **The approved mockup is `clock-mockup-v16.html`** — open it in a browser; it is interactive. Match it for layout, spacing, type, colors, gestures, and page structure. This spec wins on data and behavior; the mockup wins on visuals.

## 0. Non-negotiables
- Exactly one `index.html` (React 18 + Babel standalone via CDN, no build step) plus `manifest.json`, a 180px icon, `README.md`, `UPDATES.md`.
- Airtable REST directly. Token entered by the user on first launch, stored in `localStorage`, never written to any file. On 401/403 clear it and re-prompt.
- The schema already exists. Never create fields or tables.
- Logs link exercises through the `Exercise` linked field only. Never free text.
- Optimistic UI on every write; one retry; on failure revert and show a red toast with Retry.

## 1. Airtable
Base `appmqPkBzBzcKlrce`.

| Table | ID | Fields used | Writes |
|---|---|---|---|
| Exercises | `tblQ39nQ8p8HRSsIC` | `Name`, `Coaching Notes` | create |
| Schedule | `tblZP0Fg1dKWJzII2` | `Day of Week`, `Order`, `Section`, `Block` `fld2GSMfso7wBQZrt`, `Week Of` `fld11dPwTJyXCHDXH`, `Exercise` (link, may be multi), `Prescription` | create/update/delete on overlay rows; delete on template rows only via explicit "Every <weekday>" |
| Logs | `tblvJWkX10l1Fs6iE` | `Exercise` (link), `Log Date`, `Log Text`, `Done` `fldWk8GzrsW15fNhv` (checkbox) | create/update/delete |
| Races | `tblusBLP1tM1sR7DH` | `Name`, `Date`, `City/State`, `Mi`, `Elev`, `Cutoff (hr)`, `Status`, `Training Value` | create/delete |
| Day Notes | `tblxTF4kmjHnGv4fx` | `Date` `fldTYmmQOFCd4x8V4`, `Notes` `fldqHY40fQEWHG43i` | create/update |
| Settings | `tbl6UNTYuMVQ1u4mj` | — | ignore (legacy) |

**One-time migration (run once, at the start of the build, before any UI):** set `Done = true` on every existing Logs record (446 as of 2026-09-24). Batch 10 per request. Confirm the count in the console.

### Week resolution
`weekStart` = Monday of the selected date. If any Schedule rows have `Week Of = weekStart`, those rows are the week (overlay). Otherwise template rows (`Week Of` blank). A day = rows matching `Day of Week`, sorted by `Order`. One line per linked exercise when `Exercise` holds several.

### Copy-on-write
The first edit to any day of a template week copies every template row for that week into overlay rows (`Week Of = weekStart`, same Day of Week / Order / Section / Block / Exercise / Prescription), then applies the edit. Batched create, 10 per request. Toast: "Customizing week of Sep 21." Templates are untouched by this path.

### Done semantics
- Toggle on, no log → create Log `{Exercise, Log Date, Log Text: "Completed", Done: true}`.
- Toggle on, log exists → set `Done = true`.
- Toggle off, `Log Text = "Completed"` → delete the log.
- Toggle off, any other text → set `Done = false`, keep the log. Row shows the toggle off with the orange dot in the knob.
- Saving a result → create or update Log text and set `Done = true`.
- Skip → Log Text `Skipped`, `Done = true`, row dimmed.

### Notes storage
Day Notes `Notes` is plain text, one entry per line. Line 1 may be `Readiness: Green|Yellow|Red` (stripped from display, rendered as the segmented control). An exercise note saved from the exercise page is stored as `<Exercise Name>: <text>`. Pain/RPE are appended to `Log Text` as ` · pain N · RPE N` and parsed back out for display (pain ≥ 2 amber, 3 red).

## 2. Screens

Tab bar (floating pill, Clock-style, orange active): **Today · Exercises · Journal · Races · Add**. Add is orange and context-aware (see §3).

### Today
- Top: ◐ theme pill (left), **Edit** pill (right).
- Title = weekday with `‹ ›` arrows; subtitle = date · race countdown in orange (next race with `Date ≥ today`, `Status ≠ Completed`). Horizontal swipe on empty list space = ±1 day.
- **Notes row** first: "Notes" at row size, readiness dot + word + note count on the right. Tap → Notes page.
- **Exercise rows**: name only, 28pt weight 400, green toggle on the right. No section headers, no subtitles. Tap name → Exercise Page. Tap toggle → Done semantics.
- **Swipe left** → red Delete (park at 60px, delete past 150px; axis-locked, cancelled touches snap back). After delete: 5-second toast "Removed X" with **Undo** and, for exercises, a gray **Every <Wkd>** option that promotes the delete to the template row.
- **Press-and-hold 400ms** → row lifts; drag to reorder; orange insertion line; release writes `Order` 10, 20, 30… on the day's overlay rows.
- **Edit** pill → Clock edit mode: minus circles (delete), ☰ grips (reorder), toggles hidden. Same writes as the gestures.

### Exercise Page (from Today)
`‹ <Weekday>` back, `n of N`, **Skip** pill. Title, subtitle (Block · pairing text from prescription if it contains "paired with"). Panels: Prescription + Last result (date); Result input with orange **Save**, quick chips (last result · last with first `\d+ lb` +5 · Skipped), **Note** line (saves to Day Notes prefixed with exercise name), Pain 0–3 and RPE 6–10 segmented controls; Coaching notes; History (5, Show all). Floating Previous / Next bar; horizontal swipe does the same. Save does **not** auto-advance.

### Notes Page
`‹ <Weekday>` back, date. Readiness segmented control (Green / Yellow / Red, colored when selected), add field with **Add**, today's notes with times (from record modified time if available, else blank), "Earlier this week" from the other days of the same week (read-only). Swipe-left on a today note deletes it (Undo toast).

### Exercises
Search field, alphabetical name-only rows, no swipe. Tap → Library Page: same panels as the Exercise Page minus the Result group, plus an **Add to a day** row → day picker (writes a Schedule row per §3). Archived exercises (no Schedule row) at the bottom, dimmed.

### Journal
One row per date with ≥1 log, newest first: weekday large, date beneath, log count on the right (today in orange). Tap → Journal Day Page: readiness in subtitle, Notes panel, one cell per log (exercise → result, pain/RPE inline, Skipped dimmed), Previous / Next day bar, **Open day** link (Today tab on that date).

### Races
Name-only rows with days-until on the right (next race orange, A-race white, others gray; completed at the bottom dimmed with a green ✓). Sorted by date. Swipe-left → Delete with Undo toast. Tap → Race Page: facts panel (Location, Distance · Elev, Cutoff, Status), Training value panel, Previous / Next race bar. No Plan/Countdown panels in v1 (mockup shows them as an idea only).

### Theme
◐ pill toggles dark/light. Dark is the Clock palette in the mockup's `:root`; light is the `[data-theme="light"]` block. Follows system on first launch, then persists.

## 3. Add (context-aware)
| On | Sheet | Writes |
|---|---|---|
| Today | Exercise picker: search, results with home day · prescription, "Create '<query>'", Apply-to (This week / Every <Wkd>), **Plan next week** row | overlay row (`Order` = max+10, `Section` "Main Session", `Prescription` = the exercise's template prescription if one exists) or template row; Create → Exercises record first |
| Exercises | New Exercise: name, coaching notes, Schedule (Library only / Add to a day…) | Exercises record; optionally a Schedule row via the day picker |
| Races | New Race: name, date, city/state, distance, elevation, cutoff, Status (Registered / Waitlist / Planned), training value | Races record |

**Plan next week:** if next week has no overlay, copy this week's resolved rows to `Week Of = next Monday`, jump the Today view to next Monday, toast "Week of Sep 28 ready to edit." If it exists, just jump.

## 4. Loading
- Launch: Exercises, Schedule (all rows), Races. Refresh on foreground if > 1 hr.
- Per date: Logs (`{Log Date}='YYYY-MM-DD'`) + Day Notes; cache per date; prefetch ±1 day.
- Last result / history per exercise: `filterByFormula` on exercise name, sort desc, `maxRecords` 1 or 5; cache per exercise per session.
- Journal: Logs sorted desc, `pageSize=100`, paginate on scroll; Day Notes per rendered date.

## 5. Out of scope
Search; auto-advance after save; session timer; race editing (use Airtable); prescription/coaching-note editing; offline queue; iPad.
