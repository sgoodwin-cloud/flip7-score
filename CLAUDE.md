# Flip 7 Score — notes for Claude

Read `DESIGN.md` first. It holds the release process, code structure, the Swipe Snake lessons and the milestone status. The full design doc (scoring rules, screens, milestones) lives in Stuart's Claude doc linked from `DESIGN.md`.

- Keep the app fully offline: no runtime network calls, fonts bundled in `fonts/`.
- On every release, bump `APP_VERSION` in `index.html` and `VERSION` in `sw.js` together, to the same number: the service worker refuses to install if they differ.
- Add any new file to `CORE` in `sw.js`.
- Each milestone goes to a test branch first; merge to `main` only after Stuart has play-tested it.
- Never `preventDefault()` touches on buttons; keep the global `[hidden]` rule.
- Saves carry a version number (`SAVE_VERSION`); upgrade old saves in `migrate()`, never wipe them.
- Rules and FAQ text in the app are written in our own words, not copied from the rulebook.
- Update `DESIGN.md` (status, decision log, "Last updated") in the same commit as the change.
- UK spelling throughout. Commits as Stuart Goodwin.
