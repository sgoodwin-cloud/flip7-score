# Flip 7 Score — design and release notes

Last updated: 21 Oct 2026 (v5)

The full design doc (goals, scoring rules, rules/FAQ plan, screens, data model, milestones, Swipe Snake lessons) is Stuart's Claude doc "Flip 7 Score Tracker — Design Doc": https://claude.ai/code/artifact/daefd7ee-eb95-4476-ac02-92a735e470de

This file is the repo-side summary for continuity across chat sessions.

---

## 1. Status

| Milestone | Status |
|---|---|
| 1. Basic scorer + rules and FAQ | Done (artifact preview: https://claude.ai/artifact/4MR4BWP7uaTz1cpkP2Ay5q) |
| 2. Live on your phone (Pages, icon, offline, install tip) | Done (v1 live, installed on Stuart's iPhone) |
| 3. Fixing mistakes and history | Done (v3) |
| 4. Polish and extras | Not started |
| 5. Card-tap calculator (optional) | Parked: decide after a few real games |

Live app: https://sgoodwin-cloud.github.io/flip7-score/

---

## 2. Files

| File | What it is |
|---|---|
| `index.html` | The whole app: styles, screens, script. One file, no build step |
| `sw.js` | Service worker: offline cache and updates. `VERSION` lives here |
| `manifest.webmanifest` | Home screen name, colours, icons |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | App icons |
| `fonts/` | Bundled design-system fonts (woff2, Latin subset) |

### Code structure inside `index.html`

- `<style>`: design-system tokens on `:root` (Rulebook light theme only; the Night theme is deliberately left out), then `fs-` components from the Flip Score design system, then screen styles.
- Screens are `<section>`s toggled with `hidden`. Main screens follow `state.screen`: `scrSetup`, `scrBoard`, `scrWinner`. Overlays sit on top via the `overlay` variable (not saved): `scrRules`, `scrSheet` (editable scoresheet), `scrPast` (past games), `scrPastGame` (one past game, read-only). Sheets: `entrySheet` (keypad), `confirmSheet`, `shareSheet` (link, Copy link, Send…, install steps).
- Script sections: state and saving → scoring helpers (`settle()` re-decides game over after any change; `syncHistory()` keeps past games in step) → undo → rendering (`render(scroll)`, `sheetTable()`) → setup → score entry (current round or correcting an earlier one) → round flow → confirm sheet → scoresheet and past games → rules and FAQ content → share → install tip → boot.
- The update banner and service worker registration are a separate script at the bottom.

### Save format (localStorage key `flip7-score`), version 2

```json
{ "v": 2, "screen": "setup|board|winner",
  "draft": ["names in seat order"],
  "savedNames": ["most recent first, up to 16"],
  "game": { "id": "...", "startedAt": 0, "players": [{"id": "...", "name": "..."}], "target": 200, "seq": 7,
            "rounds": [{ "<playerId>": {"pts": 26, "bust": false, "t": 3} }],
            "current": { "<playerId>": {"pts": 0, "bust": true, "t": 7} },
            "finished": false },
  "history": [{ "id": "...", "startedAt": 0, "endedAt": 0, "target": 200, "players": [], "rounds": [] }] }
```

Totals are always worked out from `rounds`, never stored. `t` is the order scores were entered in, which Undo works back through (corrections keep their original `t`). `history` is newest first, up to 60 games; a game is added when it finishes and removed again if Undo or a correction reopens it. Abandoned games (New game mid-way) aren't kept. `migrate()` upgrades v1 saves (adds ids, `t`, saved names, history); never wipe a save.

---

## 3. Release process

1. Build and check the change in the artifact preview first.
2. Copy it into `index.html` here (the repo version adds the `<head>`, bundled fonts, safe-area padding and the update banner).
3. Bump **both** version markers, which must match: `APP_VERSION` in `index.html` and `VERSION` in `sw.js` (`flip7-score-vN`).
4. Add any new file to `CORE` in `sw.js`.
5. Push to a test branch (e.g. `m3`). Stuart play-tests; then merge to `main`. Pages republishes in 1–2 minutes.
6. On the phone: open the app with signal → "Update ready" banner → tap → "Updating…" → reloads on the new version.

---

## 4. Lessons carried over from Swipe Snake

- On a phone, GitHub folds the Settings side menu away; go straight to `github.com/<owner>/<repo>/settings/pages`.
- Claude can't create repos or switch on GitHub Pages. Stuart does those in GitHub (Pages via Safari: Settings → Pages; the iOS app has no Pages setting).
- `[hidden] { display: none !important; }` is global, so no layout rule can keep a hidden screen visible.
- No blanket `preventDefault()` on touches. Buttons use `touch-action: manipulation` to stop double-tap zoom.
- The service worker installs with `cache: 'reload'` so updates never store stale files.
- The update banner reloads only once the new worker reports `activated` (timer is a last resort).
- `controllerchange` only reloads the page when a worker was already in charge (an update). Otherwise the very first visit reloads itself a second after opening and can lose what was just typed (fixed in v3).
- The first time the update mechanism itself changes, the phone may need a full close and reopen.
- Home screen apps keep their own storage, separate from Safari. Deleting the icon deletes saved games.
- No browser `alert()`/`confirm()`: confirmations are built into the page.
- With `black-translucent`, iPhone status-bar text is white, so the band behind it (`body::before`) is painted navy, not cream.

---

## 5. Decision log

| Date | Decision |
|---|---|
| 18 Oct 2026 | GitHub Pages web app, one phone keeps score, no live sync |
| 18 Oct 2026 | Rules and FAQ in the app from Milestone 1, written in our own words |
| 18 Oct 2026 | Styled with Stuart's "Flip Score" design system; fonts bundled for offline |
| 18 Oct 2026 | Round entry by keypad in Milestone 1; card-tap calculator in Milestone 3 |
| 18 Oct 2026 | v1 pushed straight to `main`, as nothing was installed yet; test branches from Milestone 3 |
| 18 Oct 2026 | Milestone 2 done: Pages live at sgoodwin-cloud.github.io/flip7-score |
| 19 Oct 2026 | Light mode only; no dark theme (v2) |
| 19 Oct 2026 | Share button beside Rules: link, Copy link, native Send… where available, install steps (v2) |
| 19 Oct 2026 | Small non-milestone changes go straight to `main`; milestones still use a test branch |
| 19 Oct 2026 | Card-tap calculator moved to the end as optional Milestone 5; "Fixing mistakes and history" becomes Milestone 3 |
| 19 Oct 2026 | Undo works back one score at a time, then reopens the last round (even from the winner screen) |
| 19 Oct 2026 | Corrections happen on the scoresheet; game over is re-decided after every correction |
| 19 Oct 2026 | Button label "View Scoresheet" (Stuart's wording); Milestone 3 merged to `main` as v3 |
| 21 Oct 2026 | v4: update fixes from Pocket Factory. The update banner is now a bar at the top of the page (it was a floating button near the home bar, which iPhone could make untappable), and updates fetch each file at `?v=<VERSION>` and check `APP_VERSION` before installing, so GitHub Pages' CDN can't hand an update stale files |
| 21 Oct 2026 | v5: Stuart's installed copy stayed on v3. Its first v4 attempt most likely came mid-deploy and failed the version check, and iPhones can then decline to retry the same sw.js. v5 gives it a new sw.js, and installs now retry up to four times (fresh address each time) before failing |
