# Ledger

An iPhone-first PWA for daily training. It reads the plan from Airtable and
writes results and day notes back. One HTML file, React 18 and Babel standalone
from a CDN, no build step.

## Files

| File | |
|---|---|
| `index.html` | The whole app — styles, data layer and UI |
| `manifest.json` | PWA manifest |
| `icon-180.png` | Home-screen icon |
| `training-ledger-spec.md` | Build spec (data rules and edge cases) |
| `blocks-mockup-v2.html` | Approved visual mockup |
| `UPDATES.md` | Changelog |

## Getting started

Serve the folder over HTTP — opening `index.html` from the filesystem will not
work, because the app fetches across origins and registers a manifest.

```bash
python3 -m http.server 8765
```

Then open <http://localhost:8765/index.html>.

On first launch the app asks for an Airtable **personal access token**. Create
one at <https://airtable.com/create/tokens> with:

- scopes `data.records:read` and `data.records:write`
- access to the **Training** base (`appmqPkBzBzcKlrce`)

The token is kept in `localStorage` on that device and is never written to this
repo. If Airtable returns 401 or 403 the token is cleared and the app asks
again.

To install on an iPhone: open the URL in Safari, then Share → Add to Home
Screen. It runs full-screen with a black-translucent status bar and respects the
safe-area insets.

## Deploying

The app is static, so GitHub Pages serves it as-is from `main`. Settings →
Pages → Deploy from a branch → `main` / root. The token is entered per device,
so a public Pages URL exposes no credentials.

## Airtable

Base `appmqPkBzBzcKlrce` ("Training"). The schema already exists; the app never
creates fields or tables, and never writes to Exercises, Schedule or Races.

| Table | Fields used | Access |
|---|---|---|
| Exercises | `Name`, `Coaching Notes` | read |
| Schedule | `Day of Week`, `Order`, `Section`, `Block`, `Week Of`, `Exercise`, `Prescription` | read |
| Logs | `Exercise` (link), `Log Date`, `Log Text` | read / write |
| Races | `Name`, `Date`, `Status` | read |
| Day Notes | `Date`, `Notes` | read / write |

Logs always link an exercise through the `Exercise` field, never free text.
Day Notes holds one record per date, notes one per line, upserted by date.

### How a day is resolved

1. Take the Monday of the requested week. If any Schedule rows carry that
   Monday in `Week Of`, **only** those rows are used for the whole week — a
   custom week fully replaces the template. Otherwise the template rows (blank
   `Week Of`) are used.
2. Keep the rows whose `Day of Week` matches, sorted by `Order`.
3. Group them into cards by `Block`, falling back to `Section`. Cards are
   ordered by the lowest `Order` they contain, and the Notes card is always
   last.
4. A row whose `Exercise` links several records renders one line per exercise,
   each with its own checkbox, all sharing the row's `Prescription`.
5. If the first row of a card links a single exercise whose name matches the
   block name, that row's `Prescription` becomes the card's prose description
   and the row is not drawn as a line of its own.

`window.LEDGER.day('2026-09-21')` prints the resolved cards for any date to the
console, which is the quickest way to check a change to the plan.

## Configuration

Near the top of `index.html`:

- `BASE`, `TBL` — base and table ids.
- `BENCHMARKS` — the four exercise names on the PRs tab. **These are
  placeholders.** They must match `Exercises.Name` exactly. The trend assumes a
  higher leading number is better, which does not hold for a timed trial where
  faster is better.
- `STALE_MS` — how old the Schedule may be before it refetches when the app
  returns to the foreground (one hour).

## Behaviour worth knowing

- Writes are optimistic and retried once. After a second failure the change is
  rolled back and a red **Retry** appears on the affected line. Input is never
  blocked.
- Ticking an exercise with no result logs `Completed`. Unticking deletes that
  log outright; unticking a real result asks first.
- **Log Result** saves only fields that differ. Blank fields are left alone, so
  it will not wipe an existing result.
- Theme follows the system on first launch, then remembers your choice.

## Not in v1

Editing Exercises, Schedule or Races in the app; iPad layout; an offline queue
beyond the single retry; charts beyond the PR trend bars; export.
