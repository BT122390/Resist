# Resist — Claude Code handoff (v3, final design)

Put these two files in the repo folder and delete every earlier spec and mockup (`training-ledger-spec*.md`, `gmail-mockup*.html`, `blocks-mockup*.html`, `anydo-mockup*.html`, `bearlog*`, `ledger*`, `clock-mockup-v1` through `v15`):

- `resist-spec-v3.md` — the complete specification. Wins on data and behavior.
- `clock-mockup-v16.html` — the approved mockup. Interactive; open it in a browser. Wins on visuals.

## First prompt to Claude Code

---

The design for this app has been finalized and replaces anything you built before. Delete all existing UI code. Keep only the Airtable data-layer code if it exists (fetching, Week Of resolution, copy-on-write), and verify it against the new spec.

Read `resist-spec-v3.md` fully. Open `clock-mockup-v16.html` in a browser and go through every screen and gesture: Today with toggles, swipe-to-delete, press-and-hold reorder, the Edit pill, the day arrows; the Exercise Page; Notes page; Exercises tab and Library page; Journal and its day page; Races and the race page; the three Add sheets; the Undo toast; light and dark themes. The app must match it. The spec wins on data and behavior; the mockup wins on visuals.

Deliverables: `index.html` (React 18 + Babel standalone via CDN, no build), `manifest.json`, 180px icon, `README.md`, `UPDATES.md`. Airtable REST directly. The schema exists — do not create fields or tables. The Airtable token is entered by the user on first launch and stored in localStorage; never write it into a file. I'll paste it into the browser when you're ready.

Build everything in this session, in this order, and pause for my confirmation at the two marked points:
1. Data layer, then run the one-time migration in the spec (set Done=true on all existing Logs) and print the count. Print the resolved rows for today and next Monday to the console. **Pause.**
2. Today tab: rows, toggles with the full Done semantics, swipe-delete with Undo and Every-weekday scope, hold-to-drag reorder, Edit mode, day arrows and swipe, Notes row. Verify on the live base that a delete creates overlay rows and templates are untouched. **Pause.**
3. Exercise Page, Notes page.
4. Exercises tab + Library page, Journal + day page, Races + race page.
5. Add sheets (all three), Plan next week, theme toggle.
6. PWA manifest, README, UPDATES.md. Commit and push to main so GitHub Pages serves it.

Serve locally over the LAN so I can test on my phone during the pauses. Commit in small steps with clear messages after the second pause.

---

## What to check during the pauses

- After the migration: Airtable Logs should show every record with Done checked.
- Monday's list should have 15 rows (two Warm-Up, six Main Lifts, eight Accessory, one Finisher — no headers, just the names in Order).
- Saturday row #1 links two exercises → two rows.
- Toggle off Loaded Tibialis Raise after typing a result: log stays in Airtable, Done unchecks, orange dot appears in the knob. Reopen the day: still off with the dot.
- Swipe-delete one exercise, then check Airtable: ~70 new rows with Week Of = this Monday; the 70 template rows unchanged.
- Tap "Every Wed" in the toast: the template row for that exercise is now gone too.
- Add on Races opens the race sheet; on Exercises the new-exercise sheet; on Today the picker.
