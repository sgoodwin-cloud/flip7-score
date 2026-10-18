# Flip 7 Score — design and release notes

Last updated: 18 Oct 2026 (v1, Milestone 2)

The full design doc (goals, scoring rules, rules/FAQ plan, screens, data model, milestones, Swipe Snake lessons) is Stuart's Claude doc "Flip 7 Score Tracker — Design Doc": https://claude.ai/code/artifact/daefd7ee-eb95-4476-ac02-92a735e470de

This file is the repo-side summary for continuity across chat sessions.

---

## 1. Status

| Milestone | Status |
|---|---|
| 1. Basic scorer + rules and FAQ | Done (artifact preview: https://claude.ai/artifact/4MR4BWP7uaTz1cpkP2Ay5q) |
| 2. Live on your phone (Pages, icon, offline, install tip) | Pushed as v1; waiting on Pages switch-on and phone test |
| 3. Card-tap calculator | Not started |
| 4. Fixing mistakes and history | Not started |
| 5. Polish and extras | Not started |

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

- `<style>`: design-system tokens on `:root` (Rulebook light theme, Night dark theme), then `fs-` components from the Flip Score design system, then screen styles.
- Screens are `<section>`s toggled with `hidden`: `scrSetup`, `scrBoard`, `scrWinner`, `scrRules`. Sheets: `entrySheet` (keypad), `confirmSheet`.
- Script sections: state and saving → scoring helpers → rendering → setup → score entry → round flow → confirm sheet → rules and FAQ content → install tip → boot.
- The update banner and service worker registration are a separate script at the bottom.

### Save format (localStorage key `flip7-score`)

```json
{ "v": 1, "screen": "setup|board|winner",
  "draft": ["names in seat order"],
  "game": { "players": [{"id": "...", "name": "..."}], "target": 200,
            "rounds": [{ "<playerId>": {"pts": 26, "bust": false} }],
            "current": { "<playerId>": {"pts": 0, "bust": true} },
            "finished": false } }
```

Totals are always worked out from `rounds`, never stored. `migrate()` upgrades older `v` values; never wipe a save.

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

- Claude can't create repos or switch on GitHub Pages. Stuart does those in GitHub (Pages via Safari: Settings → Pages; the iOS app has no Pages setting).
- `[hidden] { display: none !important; }` is global, so no layout rule can keep a hidden screen visible.
- No blanket `preventDefault()` on touches. Buttons use `touch-action: manipulation` to stop double-tap zoom.
- The service worker installs with `cache: 'reload'` so updates never store stale files.
- The update banner reloads only once the new worker reports `activated` (timer is a last resort).
- The first time the update mechanism itself changes, the phone may need a full close and reopen.
- Home screen apps keep their own storage, separate from Safari. Deleting the icon deletes saved games.
- No browser `alert()`/`confirm()`: confirmations are built into the page.

---

## 5. Decision log

| Date | Decision |
|---|---|
| 18 Oct 2026 | GitHub Pages web app, one phone keeps score, no live sync |
| 18 Oct 2026 | Rules and FAQ in the app from Milestone 1, written in our own words |
| 18 Oct 2026 | Styled with Stuart's "Flip Score" design system; fonts bundled for offline |
| 18 Oct 2026 | Round entry by keypad in Milestone 1; card-tap calculator in Milestone 3 |
| 18 Oct 2026 | v1 pushed straight to `main`, as nothing was installed yet; test branches from Milestone 3 |
