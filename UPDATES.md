# Updates

## v1.1 — 2026-09-22

- The centre Log button is gone from the tab bar; **Races** takes its place.
  Races lists everything on the calendar by date, then what is done, with a
  coloured status pill — Registered reads as committed, Waitlist as still
  potential, Completed in green. Airtable has no potential/committed field, so
  the pill is driven by `Status`.
- Every block card now carries **Add** alongside Log Result and History, which
  logs any exercise against the selected day — the picker the centre button
  used to open. The three buttons are set a size smaller so they fit a phone
  without crowding.
- Exercise lines are 18px, up from 17.
- History reads `2026-09-22. 3×5 @ 185` instead of `Last: Sep 22 — 3×5 @ 185`,
  in the line dropdown, the Exercises tab, the block History sheet and the PR
  history. Since every line now carries a full date, the day's own result no
  longer needs a "Today" label.
- The header puts the date and weekday on one line: `September 22 - Tuesday`.

## v1.0 — 2026-09-21

First release. Single-file PWA against the Training base.

**Data**
- Paginated fetch of Exercises, Schedule and Races, held in memory, with a
  Schedule refresh when the app returns to the foreground after an hour.
- `Week Of` overlay: rows carrying a week's Monday replace the template rows
  for that whole week.
- Cards group by `Block`, falling back to `Section`, ordered by the lowest
  `Order` in the card.
- A Schedule row linking several exercises renders one line each.
- A leading row named after its block supplies the card's description instead
  of a line.

**Week**
- Header with race pill, date picker and theme toggle; week strip showing today
  in red, the selected day in a pill and a green dot for days with logs.
- Swipe the card area to move a week.
- Block cards with done / total and a per-line dropdown showing the
  prescription, the current result and the two before it.
- Checkbox logs `Completed`, or clears a result after confirming.
- **Log Result** saves a whole block at once, writing only what changed.
- **History** lists the last five results per exercise in the block.
- Notes card with Add Note and swipe-left to delete.

**Journal**
- Logs newest first, 100 per page, infinite scroll.
- Grouped by that date's blocks, with an Unscheduled card for anything off
  plan, `Skipped` entries struck through and the day's note underneath.
- Tapping a line opens the Week tab on that date with the line expanded.

**Exercises**
- Searchable library, scheduled first and a greyed "Not scheduled" card below.
- Per exercise: prescription and day, coaching notes and the last three
  results, with Show more fetching ten.

**PRs**
- One card per benchmark with the latest result, its change and a trend from
  the leading number or clock time. Tap for the full history.

**Log button**
- Log any exercise against any date, or add a day note.

**Notes**
- Writes are optimistic with one retry, then a red Retry on the line.
- The token is entered on first launch and kept in `localStorage`; a 401 or 403
  clears it and re-prompts.
- The theme attribute sits on `<html>` rather than the phone element, so
  inherited text colour resolves correctly in dark mode. The mockup had this
  the other way round and lost any text that inherited from `body`.
- `BENCHMARKS` in `index.html` is a placeholder list — the mockup's four
  benchmarks do not exist in the base.
