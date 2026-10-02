# Band App — notes for AI assistants

A living log of how this codebase works and the practices that have worked well. **Update it when you learn something non-obvious.**

## Shape of the project
- **Everything is in `index.html`** (about 9k lines: CSS, then HTML screens, then one big `<script>`). There's no build step, no bundler, no npm deps and no tests. Deployed as a static page.
- Data lives in **Google Drive** (`Drive.*` helpers). Shared JSON goes in the `_meta` folder (`songs.json`, `snippets.json`, tabs, comments). `localStorage` holds per-device preferences only (theme etc.).
- Globals: `S` (app state), `$()` = `getElementById`, `showToast()`, `showScreen(id)`, `showLoading()/hideLoading()`, `_simplePickerModal()`.
- Feature state objects: `TE` (tab editor), `SnipR` (snippet recorder), `_LT` (loop & trim), `_bpm` (BPM tapper), `DB` (doodle board), `_PLK` (song plucker).
- Find sections by their banner comments (`/* ── METRONOME ──`, `TAB EDITOR STATE`, etc.) with grep. Don't read the file top to bottom.
- Primary users are on **phones** (Android Chrome and iPhone Safari), mostly touch. The tab editor is landscape-only.

## Working methodology
1. **Ask before building** when a request has UX ambiguity (what a second or third tap does, edge cases like empty slots). Patrick likes being asked, and answers quickly.
2. **Grep first, then read narrow ranges.** Map every reader and writer of a data field before changing its shape (e.g. `grep -n "\.fret\b"` found every place tab notes are rendered or exported).
3. **Keep data changes backward compatible.** Add optional fields and don't rename. Old saved data must still load (e.g. tab notes gained optional `mod`/`modPos`, and `fret` may be `null`).
4. **Dedupe when you touch duplicated code.** `tabViewTab` and `downloadTab` had copy-pasted ASCII export, so both now use `_tabAsciiText()`.
5. **Verify before calling it done:**
   - Syntax: extract the inline `<script>` blocks and run `node --check` on each.
   - Pure logic (formatters, exporters): pull the functions into a scratch `.js` and run them in Node with sample data.
   - Visuals: headless Edge (`msedge.exe --headless=new --window-size=900,420 --virtual-time-budget=4000 --screenshot=out.png file:///…/test.html`). Use a scratch copy of `index.html` with an injected `<script>` that opens the screen and seeds state. Drive auth isn't needed for most UI.
   - Anything with the mic, real audio timing or touch gestures can't be verified headlessly. Say so and ask Patrick to test on a phone.
6. **When editing via Python/heredoc scripts**, watch out for backslash and `\n` escapes being mangled in the shell. Prefer the Edit tool for strings containing `\`. Re-grep after scripted edits.
7. **Patrick tests on his phone against the version pushed to GitHub.** Local edits aren't live until they're committed and pushed. When asking for a phone test, say so explicitly and offer to push. A "still broken" report can just mean the fix never shipped.
8. Commit only when asked. Commit messages are short, imperative and list the features (see `git log`).

## Hard-won lessons
- **Mic recording must disable voice processing:** `getUserMedia({ audio: { echoCancellation:false, noiseSuppression:false, autoGainControl:false } })`. The defaults are tuned for speech and make music sound warbly or underwater (this was the "garbled snippet" bug on Android). Any future mic feature (e.g. the tuner, see `docs/TUNER_SPEC.md`) needs the same.
- **Never play audible clicks while the mic is recording.** They bleed into the take. The snippet recorder's metronome is now a visual flash only.
- **Audio timing:** schedule sounds with Web Audio lookahead (`setInterval` about 25 ms, schedule about 120 ms ahead on `ctx.currentTime`). Sync visuals by queueing the scheduled times and checking them in `requestAnimationFrame`. Don't trust `setTimeout` for beats. Close `AudioContext`s when done, because mobile browsers limit how many can be open.
- **Async save buttons:** every early `return` and every `catch` must reset the "saving" guard and re-enable the button, or the UI gets stuck on "Saving…".
- **Touch long-press on buttons:** use pointer events plus `touch-action:none`, `-webkit-touch-callout:none`, `user-select:none`, and `preventDefault` on `contextmenu`. Otherwise Android cancels the press or pops a menu.
- **Popups positioned with `position:fixed`:** append them to `document.body` so a transformed ancestor can't break positioning.
- **Canvas `const` scoping:** a `const` declared inside an `if` and used after it throws a ReferenceError mid-render. It did, silently killing tab auto-scroll.

## Feature notes
- **Tab editor:** a note is `{ slot, string, fret, mod?, modPos? }`. `fret:null` means a symbol-only note (e.g. a muted `x`). Symbol mode (`TE.symOn`, `TE.sym`): tapping a string cycles `5 → 5/ → /5 → 5`, and a lone symbol toggles. On the canvas the fret number stays centred on the slot and the symbol hangs off one side. In the ASCII export (`_tabAsciiLines`), slot widths are set by the fret numbers only, and **symbols replace a dash in the gap** (`13p11`, `/5`) and never widen the column. Patrick needs timing to line up across strings. The one exception: a column widens by one char when a post-symbol and the next slot's pre-symbol would land on the same char. `maxW` wraps into rows (34 chars for WhatsApp portrait). Sharing uses `navigator.share` with the text wrapped in triple backticks (WhatsApp monospace), falling back to `wa.me`.
- **BPM tapper:** it auto-resets after 4 s idle. While its metronome runs, only the tap run resets, so the tempo stays. The metronome follows re-taps live and stops on Close or Reset.
