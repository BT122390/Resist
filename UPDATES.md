# Updates

## v3.0 — 2026-09-24

Rebuilt on the Clock-style design. The v2 Gmail-style UI is gone; the Airtable
layer carried over and was checked against the new spec.

**Data**
- One-time backfill checks `Done` on every log that predates the field (446 of
  446). It sits behind a button, and only writes rows still unchecked.
- Week resolution: rows carrying a week's Monday in `Week Of` replace the
  template for that week.
- Copy-on-write on the first edit to a template week, batched ten per request,
  now single-flight so two quick edits cannot copy a week twice.
- Pain and RPE ride on `Log Text`; readiness is the first line of a day's notes.

**Today**
- Clock list: theme and Edit pills, weekday with arrows, date and race
  countdown, Notes row, then one green toggle per exercise.
- Full `Done` semantics, including keeping a real result when the toggle comes
  off and showing it as an orange dot in the knob.
- Swipe to delete with Undo and an "Every &lt;Wkd&gt;" promotion to the template.
- Press and hold to reorder; Edit mode for the same two actions.
- Day arrows, and a horizontal swipe on empty space.

**Exercise Page**
- Prescription and last result, a result field with quick chips (last result,
  that result with its first weight +5 lb, Skipped), a Note line that writes to
  the day's notes prefixed with the exercise name, Pain and RPE, coaching notes
  and history. Saving sets `Done` and stays put.

**Notes page**
- Readiness, an add field, the day's notes newest first, and the rest of the
  week read-only. Swipe a note aside to delete it.

**Exercises, Journal, Races**
- Library behind a search field, unscheduled exercises dimmed at the bottom, and
  a Library page with an Add-to-a-day picker.
- Journal by date, newest first, a hundred at a time; a day page with readiness,
  notes and every log.
- Races with days-until, finished ones ticked below; swipe to delete with Undo;
  a race page with the facts and training value.

**Add**
- Context-aware: the exercise picker on Today, New Exercise on Exercises, New
  Race on Races. Plan next week copies this week forward; Reset week drops an
  overlay after ten seconds of Undo.

**Notes on the spec**
- Airtable's race columns are `Miles` / `Elevation` / `Cutoff Hours`, not the
  spec's `Mi` / `Elev` / `Cutoff (hr)`.
- Race status offers `Registered` and `Waitlist`; the spec's `Planned` is not a
  choice on the field.
- Per-note times exist only for notes typed in the current session — a day's
  notes share one record and one modified time.
- The mockup's Plan and Countdown panels on the race page are left out, as the
  spec marks them out of scope for v1.
