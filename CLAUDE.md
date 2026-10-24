# Band App — notes for AI assistants

A living log of how this codebase works and the practices that have worked well. **Update it when you learn something non-obvious.**

## Shape of the project
- **Everything is in `index.html`** (about 9k lines: CSS, then HTML screens, then one big `<script>`). There's no build step, no bundler, no npm deps and no tests. Deployed as a static page.
- Data lives in **Google Drive**. Shared JSON goes in the `_meta` folder (`songs.json`, `snippets.json`, `tabs.json`, `comments.json`, `structures.json` (legacy, read-only now), `feather_images.json`, `dev-notes.json`).
- **Never read or write `_meta` JSON directly.** Use `Meta.load(name)` and `Meta.save(name, () => array)`, or the wrappers (`saveSongsJson`, `saveSnippets`, `saveTabs`, `saveCommentsJson`, …). `Meta.save` re-reads Drive and three-way merges our edits (`_merge3`) into whatever is there, so two bandmates editing at once don't erase each other. It also queues saves per file and shows a toast if a save fails.
- Drive calls go straight to the REST API through `Drive._fetch` (no gapi client). It gets the token from `Auth.get()`, and on a 401 it waits for a reconnect and retries.
- **Auth:** the token is kept in `localStorage` (`bandapp_token`) for its hour, so reopening the app needs no popup. Phones block popups that don't come from a tap, so: never call `requestAccessToken` outside a click handler. When the token is gone, returning users get a one-tap "Continue as X" button, and a mid-session expiry shows the `#reconnect-bar`.
- **Device caches:** audio, images and snippets never change once uploaded, so `Drive.fetchBlob` keeps them in Cache Storage (`BlobCache`, 300 MB, least recently used goes first). Waveform peaks and lengths live in `localStorage` (`pk2_<fileId>`, `ng_durs`). The track index lives in `localStorage` (`ng_trackIndex`). It's used straight away and refreshed in the background every 2 minutes.
- Globals: `S` (app state), `$()` = `getElementById`, `esc()` (escape text for `innerHTML`; use it for anything a person typed), `showToast()`, `showScreen(id)`, `showLoading()/hideLoading()`.
- Shared UI helpers: `_promptModal`, `_confirmModal`, `_pickSongModal` (searchable song/idea picker with "+ New idea"), `_simplePickerModal` (plain list), `_attachSwipeConfirm` (swipe-right-to-delete on any row).
- Feature state objects: `TE` (tab editor), `SnipR` (snippet recorder), `_LT` (loop & trim), `_bpm` (BPM tapper), `DB` (doodle board), `_PLK` (song plucker), `SessPlayer` (session player).
- Find sections by their banner comments (`/* ── METRONOME ──`, `TAB EDITOR STATE`, `RECORDING FILENAMES`, etc.) with grep. Don't read the file top to bottom.
- Primary users are on **phones** (Android Chrome and iPhone Safari), mostly touch. The tab editor is landscape-only.

## Data model in brief
- **Songs and Ideas** are both entries in `songs.json`. An idea has `type:'egg'` (kept for old data; the UI says "Idea"). An idea is a full song page with a required working title. "Hatch" deletes `type` and can rename it (cascades through `renameTrack`). `_songLists()` gives `{ songs, ideas }`, A–Z and deduped.
- **Recordings** are mp3s in the dated session folders. `parseRecordingName()` reads `Title - 2 - 2025-03-01[ - extra].mp3` (also `V2`). **Never split filenames on every "-"**: it chops the date and breaks titles with hyphens. `_versionsFor(entry, name)` = index entries for that name, plus an idea's legacy `eggFileId`.
- **Lengths** are `song.durations[fileId]`. They're only ever set by decoding that exact file (`analyseRecording` → `rememberDuration`). The old `versionDurations` field was polluted and is ignored.
- **Comments:** there's one shared list, `_cdAllComments`. `S.tp.comments` and `SessPlayer.comments` are getters that filter it. Don't keep separate copies again.
- **Structure:** the doodle board (`song.doodle`) is the only structure tool. Old `structures.json` entries only seed a song's doodle the first time (`_legacyStructureBars`).
- Lyric annotations store the `text` they cover, and `_anchorAnnotations` re-finds it after the lyrics are edited.

## Working methodology
1. **Ask before building** when a request has UX ambiguity (what a second or third tap does, edge cases like empty slots). Patrick likes being asked, and answers quickly.
2. **Grep first, then read narrow ranges.** Map every reader and writer of a data field before changing its shape (e.g. `grep -n "\.fret\b"` found every place tab notes are rendered or exported).
3. **Keep data changes backward compatible.** Add optional fields and don't rename. Old saved data must still load (e.g. tab notes gained optional `mod`/`modPos`, and `fret` may be `null`).
4. **Dedupe when you touch duplicated code.** `tabViewTab` and `downloadTab` had copy-pasted ASCII export, so both now use `_tabAsciiText()`. The same went for swipe handlers, dialogs and WAV encoding (`_encodeWav`).
5. **Verify before calling it done:**
   - Syntax: extract the inline `<script>` blocks and run `node --check` on each.
   - Pure logic (formatters, exporters, `_merge3`, `parseRecordingName`): pull the functions into a scratch `.js` and run them in Node with sample data.
   - Visuals and flows: headless Edge on a scratch copy of `index.html` with the Google script removed and an injected harness that replaces `Drive.list/query/readJson/writeJson/fetchBlob` with in-memory fakes, then runs a scenario from `location.hash`. Write results into a `<pre>` and read them back with `--dump-dom`.
   - Headless Edge has a minimum window width of about 500 px (narrower screenshots are just cropped), and `--virtual-time-budget` races ahead of real `decodeAudioData`, so waveform/length results can look missing when they aren't.
   - Anything with the mic, real audio timing or touch gestures can't be verified headlessly. Say so and ask Patrick to test on a phone.
6. **When editing via Python/heredoc scripts**, watch out for backslash and `\n` escapes being mangled in the shell (an `aren\'t` lost its backslash this way). Write the Python to a file instead of a heredoc, or use the Edit tool for strings containing `\`. Re-grep after scripted edits.
7. **Patrick tests on his phone against the version pushed to GitHub.** Local edits aren't live until they're committed and pushed. When asking for a phone test, say so explicitly and offer to push. A "still broken" report can just mean the fix never shipped.
8. Commit only when asked. Commit messages are short, imperative and list the features (see `git log`).

## Hard-won lessons
- **Shared files were overwritten whole.** Each phone loaded `songs.json` once and later wrote its whole copy back, so one person's edit erased another's ("my change vanished"). Hence `Meta.save` with merge. Saves also used to ignore HTTP errors, so failures looked like successes.
- **The shared `<audio>` element lies about which file it's describing.** A `loadedmetadata` listener added after `play()` resolves fires for the *next* file, which gave tracks the previous/next track's length, and those values were saved. Measure lengths from the decoded file, and guard async loads with a ticket (`S.tp._loadSeq`, `SessPlayer._seq`) so a slower earlier load can't win.
- **iPhone may refuse `audio.play()` after an await** (no longer inside the tap). Treat that rejection as "paused", not "failed to load".
- **Mic recording must disable voice processing:** `getUserMedia({ audio: { echoCancellation:false, noiseSuppression:false, autoGainControl:false } })`. The defaults are tuned for speech and make music sound warbly or underwater (this was the "garbled snippet" bug on Android). Any future mic feature (e.g. the tuner, see `docs/TUNER_SPEC.md`) needs the same.
- **Never play audible clicks while the mic is recording.** They bleed into the take. The snippet recorder's metronome is now a visual flash only. A snippet stores `bpm` only if its metronome was switched on.
- **Audio timing:** schedule sounds with Web Audio lookahead (`setInterval` about 25 ms, schedule about 120 ms ahead on `ctx.currentTime`). Sync visuals by queueing the scheduled times and checking them in `requestAnimationFrame`. Don't trust `setTimeout` for beats. Close `AudioContext`s when done, because mobile browsers limit how many can be open.
- **Async save buttons:** every early `return` and every `catch` must reset the "saving" guard and re-enable the button, or the UI gets stuck on "Saving…".
- **Debounced text saves** (lyrics/notes, 1.5 s) must be flushed before the page changes song (`flushPendingEdits`), or the edit is lost.
- **Touch long-press on buttons:** use pointer events plus `touch-action:none`, `-webkit-touch-callout:none`, `user-select:none`, and `preventDefault` on `contextmenu`. Otherwise Android cancels the press or pops a menu.
- **Popups positioned with `position:fixed`:** append them to `document.body` so a transformed ancestor can't break positioning.
- **Google sign-in scopes are optional checkboxes.** Users can untick Drive on the consent screen and still get a token, and then Drive calls fail with "insufficient authentication scopes". `onToken` checks `hasGrantedAllScopes` and asks them to sign in again. The consent screen is forced only for new users or after a missing tick (`S._needConsent`).
- **Back button:** `_closeTopOverlay()` closes the topmost overlay before leaving a screen. Add new overlays to its list.
- **Canvas `const` scoping:** a `const` declared inside an `if` and used after it throws a ReferenceError mid-render. It did, silently killing tab auto-scroll.

## Feature notes
- **Tab editor:** a note is `{ slot, string, fret, mod?, modPos? }`. `fret:null` means a symbol-only note (e.g. a muted `x`). Symbol mode (`TE.symOn`, `TE.sym`): tapping a string cycles `5 → 5/ → /5 → 5`, and a lone symbol toggles. On the canvas the fret number stays centred on the slot and the symbol hangs off one side. In the ASCII export (`_tabAsciiLines`), slot widths are set by the fret numbers only, and **symbols replace a dash in the gap** (`13p11`, `/5`) and never widen the column. Patrick needs timing to line up across strings. The one exception: a column widens by one char when a post-symbol and the next slot's pre-symbol would land on the same char. `maxW` wraps into rows (34 chars for WhatsApp portrait). Sharing uses `navigator.share` with the text wrapped in triple backticks (WhatsApp monospace), falling back to `wa.me`. Leaving with unsaved notes asks first (`_tabIsDirty`).
- **BPM tapper:** it auto-resets after 4 s idle. While its metronome runs, only the tap run resets, so the tempo stays. The metronome follows re-taps live and stops on Close or Reset.
- **Snippets:** the recorder's title shows where the snippet will be saved, and tapping it changes that. From a song page it saves straight to that song. From Home it asks with `_pickSongModal`.
- **Home** has the Tools row (Record snippet, Tap BPM, Upload tracks, Pluck songs). **Feathers** is the library (snippets, lyrics, tabs, images) plus dev notes.
