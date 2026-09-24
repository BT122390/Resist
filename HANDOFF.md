# Resist — Claude Code handoff

Two files define the build:

- `training-ledger-spec-v2.md` — the complete specification. Wins on any data or behavior question.
- `gmail-mockup-v6.html` — the approved mockup. Open it in a browser (it's interactive). Wins on anything visual: layout, spacing, type sizes, colors, gestures.

Older files (`training-ledger-spec.md`, `blocks-mockup*.html`, `anydo-mockup*.html`, `bearlog*`, `ledger*`) are superseded. Ignore or delete them.

## First prompt to Claude Code

Paste this:

---

Build a single-file training PWA in this repo.

Read `training-ledger-spec-v2.md` fully — it is the complete specification and supersedes anything else in the repo. Open `gmail-mockup-v6.html` in a browser and study every screen and gesture; the finished app must match it for layout, spacing, type sizes, colors, and interactions (day pill bar, Gmail-style rows, swipe-to-remove with Undo toast, drag-handle reorder, Exercise Page with Previous/Next and swipe, Notes page, Search with week chips, Add sheet with scope toggle). The spec wins on data and behavior; the mockup wins on visuals.

Deliverables: `index.html` (React 18 + Babel standalone via CDN, no build step), `manifest.json`, a 180px icon, `README.md`, `UPDATES.md`. Talk to the Airtable REST API directly. The Airtable schema already exists — do not create fields or tables.

The Airtable personal access token is entered by the user on first launch and stored in localStorage. Never write it into any file. I will paste it into the browser when you're ready to test.

Build in this order and pause for my confirmation after step 1 and step 3:
1. Data layer — fetch Exercises, Schedule, Races, per-date Logs and Day Notes; implement week resolution (Week Of overlay vs template) and the copy-on-write materialization; print the resolved rows for today and for next Monday to the console before any UI.
2. Home: day pill bar, Notes row, exercise rows with avatar-tap-to-check, swipe-to-remove with Undo, drag reorder. Verify against the live base that a remove creates overlay rows and the template is untouched.
3. Exercise Page with result save, quick chips, Pain/RPE chips, Previous/Next.
4. Notes page with Readiness, Search (exercise results + week view), Add sheet (including Create and Plan next week), Exercises, Races, Theme.
5. PWA manifest, README, UPDATES.md.

Serve locally so I can test on my phone over the LAN. Don't commit until I confirm step 2 works on real data. After that, commit in small steps with clear messages and push.

---

## Things to watch during the build

- After the first swipe-remove, check Airtable: the week should now have ~70 rows with `Week Of` = this Monday, and the 70 template rows should be unchanged.
- Monday should render with W / M-block avatars split into Main Lifts and Accessory colors; Wednesday with S / A / B.
- Saturday row #1 links two exercises — it must render as two lines.
- Search must not match on result text (only exercise names and note text), except the fixed Pain ≥ 2 filter.
