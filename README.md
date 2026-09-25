# Resist

An iPhone-first PWA for daily training, modelled on Apple's Clock app. It
reads the plan from Airtable and writes back results, day notes, per-week
schedule edits, new exercises and races. One HTML file, React 18 and Babel
standalone from a CDN, no build step.

## Files

| File | |
|---|---|
| `index.html` | The whole app |
| `manifest.json`, `icon-180.png` | PWA manifest and home-screen icon |
| `resist-spec-v3.md` | Build spec — wins on data and behaviour |
| `clock-mockup-v16.html` | Approved mockup — wins on visuals |
| `UPDATES.md` | Changelog |

## Running it

Serve the folder over HTTP; opening `index.html` from the filesystem will not
work.

```bash
python3 -m http.server 8765
```

Then <http://localhost:8765/index.html>, or your machine's LAN address from a
phone on the same network.

On first launch it asks for an Airtable **personal access token** — create one
at <https://airtable.com/create/tokens> with `data.records:read` and
`data.records:write` on the **Training** base (`appmqPkBzBzcKlrce`). It is kept
in `localStorage` on that device and never written to this repo. A 401 or 403
clears it and asks again.

To install: open the URL in Safari, then Share → Add to Home Screen.

### One-time setup

`Done` was added to Logs after 446 records already existed. On first launch the
app offers to check `Done` on every log that has not got it, behind a button
rather than doing it automatically. It only writes rows still unchecked, so
running it twice does nothing.

## Deploying

Static, so GitHub Pages serves it from `main` as-is (Settings → Pages → Deploy
from a branch → `main` / root). The token is per device, so a public URL
exposes no credentials.

## Airtable

Base `appmqPkBzBzcKlrce`. The schema already exists; the app never creates
fields or tables.

| Table | Fields used | Writes |
|---|---|---|
| Exercises | `Name`, `Coaching Notes` | create |
| Schedule | `Day of Week`, `Order`, `Section`, `Block`, `Week Of`, `Exercise`, `Prescription` | create / update / delete |
| Logs | `Exercise`, `Log Date`, `Log Text`, `Done` | create / update / delete |
| Races | `Name`, `Date`, `City/State`, `Miles`, `Elevation`, `Cutoff Hours`, `Status`, `Race Type`, `Training Value` | create / delete |
| Day Notes | `Date`, `Notes` | create / update |

**Two places where the base differs from the spec**, both handled in code:

- The spec calls the race columns `Mi`, `Elev` and `Cutoff (hr)`; Airtable has
  `Miles`, `Elevation` and `Cutoff Hours`. The real names live in the `F` map at
  the top of `index.html`.
- The spec offers a `Planned` race status, which is not one of the field's
  choices. `RACE_STATUS` offers only `Registered` and `Waitlist`; add the choice
  in Airtable and it can go in that constant.

### How a day is resolved

1. Take the Monday of the selected date. If any Schedule rows carry that Monday
   in `Week Of`, **only** those rows are the week. Otherwise the template rows
   (blank `Week Of`) are.
2. Keep the rows matching `Day of Week`, sorted by `Order`.
3. A row whose `Exercise` links several records becomes one screen row each.

**Copy-on-write:** the first edit to any day of a template week copies every
template row for that week into `Week Of` rows and applies the edit to the copy.
The template is never written to except through an explicit "Every &lt;weekday&gt;"
action. Materialization is single-flight per week, so two quick edits cannot
copy it twice.

`window.RESIST.day('2026-09-28')` prints the resolved rows for any date.

### Done

| Action | Effect |
|---|---|
| Toggle on, no log | create a log, `Log Text` "Completed", `Done` true |
| Toggle on, log exists | `Done` true, text untouched |
| Toggle off, text is "Completed" | delete the log |
| Toggle off, real result | `Done` false, log kept — the knob shows an orange dot |
| Save a result | text written, `Done` true |
| Skip | `Log Text` "Skipped", `Done` true |

Pain and RPE are appended to `Log Text` as ` · pain 2 · RPE 8` and parsed back
out for display. Readiness is the first line of a day's notes, as
`Readiness: Green`.

## Behaviour worth knowing

- Writes are optimistic; a failure reverts the change and shows a red toast.
- Swipe a row left to park it at Delete, or past 150px to remove it. The toast
  offers Undo and, when the exercise also sits on the template, an
  "Every &lt;Wkd&gt;" promotion that removes it there too.
- Press and hold a row for 400ms to reorder; the drop renumbers the day.
- Gestures lock to an axis and a cancelled touch always snaps back, so
  scrolling never deletes.
- Theme follows the system on first launch, then remembers your choice.

## Not in v1

Search; auto-advance after saving; a session timer; editing races,
prescriptions or coaching notes in the app; an offline queue; iPad.
