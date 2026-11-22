# Plan & decision log

Patrick dips in and out of this project. This file is the memory: **what he's said, what's been
decided, what's open, what's next.** Every session: read it first, and log new decisions and wishes
before building anything. Newest entries at the top of each section. Feature specs live in
`docs/*_SPEC.md`.

## Now
- **Doodle Board / Song Canvas redesign:** see [DOODLE_SPEC.md](DOODLE_SPEC.md). Design agreed
  2026-11-20. Build order:
  1. ✅ (2026-11-20, pushed in 0b676bc, awaiting phone test) Permanent dot IDs (`doodle.barIds`), pins point at `barId`,
     deleted dot → pin floats, old `barIdx` pins migrated on open.
  2. ✅ (2026-11-21, pushed; Patrick: "looks good" on phone) Board is the song page's Structure tab: vertical rows,
     energy | dots | lanes (empty for now) | callouts (existing pins as chips). Tap ⚡ strip = hype
     1–5. Tap a dot = expand with +Note/+Tab/+New tab/+Snip/Record. Edit structure = cursor + key
     tiles. Fix-links icon + tray. Notes tab and the full-screen overlay are gone. Tabs order:
     Recordings, Lyrics, Tabs, Structure.
  3. ✅ (2026-11-21, pushed, awaiting phone test) Lyrics linking: "♪ Fit lyrics" button, lyric
     lines in the callout column, ▲▼✕ in the expanded row, faded chorus echoes, margin tags on the
     Lyrics page (tap to pick a dot), edit-mode warning, lost/floating lines in the fix tray.
     2026-11-21 follow-up (pushed): ▲▼ replaced by hold-and-drag with bump physics, on both the
     Structure tab and a new ⇅ Place mode on the Lyrics page. Verse is blue, outro warm grey.
  4. Member lanes: ✅ rests (2026-11-22, pushed), and tabs/any pin addressed to members. Still to
     do: chunk labels on lanes ("walking bass"), moments. 5. Layers (comments/tabs callouts, lyrics fade, linked track
     comment ▶). 6. Versions, then the horizontal toggle.
- After pushing step 1: everyone should hard-refresh. An old app that edits a structure changes
  `bars` without `barIds`, and pins may then land on the wrong dot.

## Working agreement
- 2026-11-21: During the doodle-board sprint, **commit and push each finished, tested change
  without asking**, so Patrick can test on his phone. (Commit messages as usual.)

## Wishlist (not scheduled yet)
- 2026-11-21: **Whole-song view:** a button that fits the full structure on one screen, a bit like
  print preview, so you can see the whole song at a glance without scrolling. Not started. Ask about
  details (what's shown, can you tap into it) before building.

## Decisions
- 2026-11-22 (Patrick, after the marker build), **direction given, details being asked:**
  - **Tabs get smaller:** the pill just says **"TAB"** (not the tab title), coloured in the band
    member's colour. **Adding a tab prompts who it's for.**
  - **Tab marker:** lose the line/T stub. Just a **tiny T in a circle on the member's lane** it's
    for, and the (TAB) pill sits **to the right of any notes** in that row.
  - **Snips are always tied to a note**, so no snip symbol (clutter). A note with a clip gets a
    **little play button in a circle at the right end of its pill**, the same diameter as the pill.
  - Adding a note should offer to **also add a snip**, which can be a **piece of a rehearsal
    recording** (from the Recordings page). The principle to embed: **"Rory, play this riff here"
    plus a button that plays that riff from a rehearsal, as easy as possible.** Don't bloat the
    code. Open: start from the structure and pick the recording bit, start from the recording and
    send it to the structure, or both?
  - **Answers:** riff clips are made **both ways, sharing one link**: a recording comment gets
    "Pin to structure", and the board's note dialog can pick one of this song's range comments.
    The note just points at the comment (`commentId`), and its ▶ plays that range. **Tabs:** Ⓣ on
    the lane only in the lyrics view; the TAB pill shows in the notes view only, with no line to
    it. A ranged tab puts its Ⓣ at the start of the bold run. **A tab is for one person or
    nobody** (nobody = no Ⓣ on any lane, just a neutral pill).
  - My calls: old `snip` pins (recorded snippets) now draw as a note pill with the ▶ end, using the
    snippet title as the text. + Snip / 🎙 Record stay in the open row.
  - Built 2026-11-22 (pushed): all of the above. "⇢ Structure" is on the song page's recording
    comments only (not the session page). Making a new riff still means adding a comment with a
    range on the Recordings tab first.
  - Useful fact: track comments already store `fileId`, `startTime`, `endTime`, `directedTo`,
    and the comments list can already play a range (`cdPlayComment`).
- 2026-11-22 (Patrick, from a phone screenshot: "ok not quite"):
  - **No `[` brackets at all**, not even for ranged pins addressed to nobody. My call: such a pin
    just says "N bars" on its chip in the notes view.
  - **The end ticks on the lanes are too much.** Instead, the bold run just stops a little short (a
    small gap) at its end.
  - **Pin markers by type** (his drawing): a **note** = a small dot on the member's lane with a
    horizontal line out to the note (in the lyrics view the line ends in a small dot). A **tab** =
    a plain line from the lane ending in a **T stub**. A **snip** = a line ending in a **little
    triangle**. Keep it uncluttered. My calls: in the notes view the stub/triangle sits just
    before the tab/snip chip. Pins addressed to nobody show the same end marker (dot/T/triangle)
    in a neutral colour, with no line.
  - **Header reveal fix:** "a swipe down to scroll to the top shouldn't reveal it, but if you are
    already at the top then swiping down will reveal it." The clamp version didn't work on his
    phone at all (momentum scrolling). New approach: once the header has scrolled out of view and
    the scroll comes to rest, it's tucked away (removed from the flow, scroll position
    compensated). A pull down that *starts* at the top brings it back.
  - Built 2026-11-22 (pushed): all of the above.
- 2026-11-22 (Patrick: "this is nice"):
  - **Column headers** at the top of the board: "Energy" written vertically over the energy
    strip, a ✎ button over the dots (edit structure; the bottom Edit button goes), and the **band
    members' names spread across the whole row with lines fanning down into their lanes**.
  - **Scrolling:** the header scrolls with the board. Scrolling back up stops just below the
    header, and a second swipe reveals it. (My reading: one stop on the way up; noted in case
    he meant something else.)
  - **The bottom Edit-structure section goes.** My calls: it only appears in edit mode (Done,
    Clear, key tiles). The fix-links icon moves into the header. "Aa" and "Fit lyrics" are dropped:
    the notes view always shows names, and lyrics are placed from the Lyrics page (Place
    auto-fits).
  - **In the lyrics view, notes/tabs still show on the member lanes:** a thick line with a
    callout line ending in a small dot. Toggling to tabs & notes inflates the dot into the
    note/tab.
  - Built 2026-11-22 (pushed): all of the above.
  - **The range bracket goes.** A note/tab covering several bars makes the addressed members'
    lanes **bold over those bars, with a little stub at the end** so adjacent ones are
    distinguishable. My call: a ranged pin addressed to nobody keeps a faint bracket, since it has
    no lane.
- 2026-11-22 (Patrick: "nice one" on the drag):
  - **Tabs can be addressed to band members too** (like notes): line from their lane.
  - **Edit the member lanes, mostly so people can see when to stop playing.** My design: in Edit
    structure the lanes column widens (initials on top) and you paint rests by tapping or dragging
    down a lane. Lanes show a solid line where that member plays and a gap where they rest. Data:
    `doodle.lanes = { member: { rest: [barId] } }`. Chunk labels ("walking bass") come later.
  - **Long lyric lines on the Structure tab are trimmed in the middle** ("start … end"), because
    the start and end are the cues.
  - **Tabs/notes left-justified**, and a **toggle between a lyrics view and a tabs/notes view**:
    tap the member lanes to flip. My calls: the lyrics view hides the pins (dots keep their rings);
    the notes view shows the pins with the lyrics faded right back underneath (as agreed on
    2026-11-20); a song with no placed lyrics always shows notes; a tapped-open dot shows
    everything; the choice is remembered per device.
- 2026-11-21 (after testing step 3):
  - **Verse and chorus colours too similar** (dark orange vs red). Verse changed to blue. Outro
    moved off slate-blue to warm grey so it doesn't clash. Barn's lane changed to cyan.
  - **Hold and drag lyric lines to a new dot** instead of ▲▼.
  - **Assign lyrics from the Lyrics page:** it's where they're in order. A place mode shows the
    structure on the left with every dot as a row (empty rows expand), and lines get dragged onto
    them.
  - **Bump physics (his spec):** lines keep their order, one per dot. Dragging the bottom line of a
    block up moves the lines above it up too, **until that block meets the block above** (no space
    left), then it stops. Release and drag again: now the merged block moves together and fills
    the space above it. This stops accidental nudges. My additions: dragging down mirrors it; a
    middle line drags the lines on its leading side; on entering place mode any unplaced lines are
    auto-placed (Fit, then just after the previous line) so every line has a dot; the same drag
    works on the Structure tab.
- 2026-11-21 (feedback after phone test of the strip; Patrick: "bars look great"):
  - **Strip "in view" marker:** dimming wasn't clear. All blocks stay full colour, with a **thin
    outlined window** around the part in view (he also floated a sideways "[" bracket).
  - **Verse dot too close to the (light) background.** Either a paler cream background or tweaked
    dot colours. I chose to tweak the dot colour (smaller change). Revisit if it's still not right.
  - **Progress bar shows the take's date and version inside it**, scrolling back and forth if it
    doesn't fit.
  - **Strip labels centred** under their section.
  - **Notes/comments can be addressed to bandmates.** A line runs from that member's lane to the
    note. (Pulls part of step 4/5 forward.)
  - **Pin a note or tab to a range** (start to end bar, e.g. a whole section). The callout shows
    the range it covers.
  - **Pins are minimised by default:** a single small dot. Tap to show the name. Applies to
    everything. He's not sure whether to also have a broad per-type filter. Controls like that go
    in the bottom bar (the Edit structure row), but it **must not take more space**. That area is
    already too tall: there's too much and uneven space under the button.
  - Built 2026-11-21: window marker, verse colour #A8501C, take label in the progress bar, centred
    labels, notes addressed To members (lane line), ranges via pin menu (Cover whole section / Cover
    up to… / Just this bar), minimised pins + "Aa" show-all-names toggle, tighter bottom bar.
  - Then do step 3 (lyrics) and commit.
- 2026-11-21: **Step 3 defaults** (asked 5 questions, no answers yet, went with my
  recommendations; revisit if they feel wrong): lyrics take the callout column and pins collapse to
  dots; a "Fit lyrics" button (only places unlinked lines, never undoes tweaks); paragraphs fill
  vocal sections in order, skipping Intro/Outro/Solo/Misc; a chorus written once is echoed (faded)
  on later choruses; move a line with ▲▼ in the expanded row; the Lyrics page shows section tags in
  the margin, with broken links marked in edit mode.
- 2026-11-21: **Mini structure strip** at the bottom of the Structure tab: one block per bar,
  **height = energy**, colour = section type, section labels (C1 etc.) underneath. My additions:
  half bars are half width, no energy = a low flat block, tap a block = scroll the board to that
  bar, and the strip highlights what's on screen. (Built 2026-11-21.)
- 2026-11-21: **On the Structure tab, the waveform is replaced by a one-line player:**
  [progress bar] [⏮] [play] [⏭]. Tap or drag the progress bar to seek. Other tabs keep the full
  waveform player. (Built 2026-11-21.)
- 2026-11-20: Step 2 layout, from my proposals (Patrick: "ok"): one row per dot, scrolling
  vertically. Half bars get half-height rows. Faint divider between sections, label on the first dot. Tap
  the ⚡ column beside a dot → 1–5 picker (+ clear). Tap a dot → its row expands to show
  attachments + quick actions (+Comment, +Tab, +Snip). Edit structure = the same vertical list with
  a cursor between rows and key tiles at the bottom, Done to return. Bottom bar: Edit structure,
  layers, fix-links icon (only when something floats).
- 2026-11-20: **Versions:** people can save versions (snapshots) of the structure and lanes.
  Details open (Q18).
- 2026-11-20: **Anyone can edit any lane.** Each chunk records who made it.
- 2026-11-20: **Member colours:** I pick them (distinct from section colours).
- 2026-11-20: **Energy shape:** taper between set points (dots without a value don't count). Each
  set point holds **flat across its own dot** before tapering, so a single quiet dot reads as a
  square step, not a spike. Before the first point, hold the first value. After the last, hold the last.
- 2026-11-20: **Energy bar** to the *left* of the dots. Set hype points (1–5) at chosen dots. The
  bar's thickness shows how hard to go for it. (Open: smooth or stepped between points.)
- 2026-11-20: **One lane per member (all five), colour-coded.** There are no member colours in the app
  yet, so they must not clash with the section colours.
- 2026-11-20: **The Structure tab *is* the board.** "Edit structure" is the deeper level (the keyboard).
  The separate full-screen overlay goes away.
- 2026-11-20: Notes tab content (`noteText`, `noteImages`): Patrick doubts there's anything worth
  keeping. Plan: leave the data in songs.json untouched (don't delete), just stop showing it.
- 2026-11-20: **Default board view = structure dots | member lanes | lyrics.** When comments (or
  other layers) are switched on, the lyrics fade far into the background *underneath* the comments,
  barely readable.
- 2026-11-20: **Linked track-comment playback:** a ranged comment plays its range. A point (or very
  short) comment plays from 2 s before to 10 s after.
- 2026-11-20: **Member lanes stay very thin until selected.** A selected lane gets thicker and shows
  its labels plus its tabs and comments.
- 2026-11-20: **Lyrics smart default:** spread a paragraph's lines evenly over its section's dots,
  where that makes sense.
- 2026-11-20: **Dots on the left**, then member lanes, then callouts.
- 2026-11-20: **Song (track) page tabs become: Recordings, Lyrics, Tabs, Structure.** The
  Structure tab *is* the doodle board (embedded), with a way to go deeper into structure editing from
  there. **The Notes tab is removed.** (Open: what happens to existing `noteText`/`noteImages`.)
- 2026-11-20: Every dot gets a permanent ID and attachments point at that ID, not at a position.
  Old pins get migrated on first open. (Built.)
- 2026-11-20: Versions save the dots, lanes and energy (optional name, who, when). Attachments
  aren't copied: they point at dot IDs, so restoring brings back whatever still matches, and the rest
  floats. Restoring auto-saves the current board as a version first.
- 2026-11-20: **Deleted dot → its attachments float.** Keep them, flag them quietly. No toasts,
  banners or badges that clutter. Expect boards to be half-finished most of the time, so floating
  items are normal, not an error. UI: **just a small "fix links" icon somewhere** (shown only when
  something is floating) that flags it and maybe opens a fix mode. No count, no toasts. Exception:
  on the **Lyrics page in edit mode**, broken or floating lyric links should be more obvious.
- 2026-11-20: **Lyrics ↔ dots: one dot per lyric row.** If a lyric spans several dots, the user
  splits it across rows. Smart default: assign lyrics **per chunk** (paragraph → section), and the user
  fine-tunes the per-dot assignment. Patrick sees the Lyrics page tying to the dots nicely, so linking
  works from both sides.
- 2026-11-20: **Member lanes are thin bars/lines running alongside the dots, not dots.** Column
  order: spine dots | member bars | callouts (lyrics, tabs, comments…). A **moment** is a single
  callout line, like a pin.
- 2026-11-20: **Default view is vertical.** Build around vertical, then make sure a horizontal
  toggle still makes sense.
- 2026-11-20: **Board comments are separate from track comments**, but a board comment can **link a
  timestamped track comment** (e.g. "Rory, play this riff here"). The board comment gets a small ▶
  that plays that stretch of the recording without leaving the board. Exact mechanics TBD.
- 2026-11-20: Restructuring the spine must never delete or misplace attachments (lyrics, tabs,
  comments, pins).
- 2026-11-20: Pinning to gaps between dots is parked. Dots only for now.
- 2026-11-20: Member lanes and layers are collapsed or hidden by default. Toggles go in the bottom bar.
- 2026-11-20: Want a vertical spine view (phone-friendly, lines up with lyrics).
- 2026-10-24: Doodle board is core. It's the only structure tool (old `structures.json` only seeds it).

## Open questions (Doodle Board)
Asked 2026-11-20, awaiting answers:
1. ~~What happens to attachments when their dot is deleted?~~ Answered: they float (see Decisions).
1–8, 10–11: answered 2026-11-20 (see Decisions).
9. Lyric links are stored by line text (like lyric annotations). If a linked line is edited, try to
   re-find it, and if that fails it floats. Not objected to so far. Treat it as the plan.
12–14: answered 2026-11-20.
15–17: answered 2026-11-20. (Proposed chunk drawing: select lane → tap start dot → tap end dot → label.)
18. Answered 2026-11-20 (see Decisions).

## Known problems
- ~~Doodle pins store `barIdx` (position), so inserting a bar shifts every later pin.~~ Fixed by
  step 1 (2026-11-20).

## Notes for later steps
- Step 2 shows existing pins as chips in the callout column. When lyrics arrive (step 3), the
  default view becomes lyrics, and pins/comments move behind the layers toggle (step 5).
- Landscape compaction of the key tiles was dropped with the old overlay. Re-check in landscape.

## Parked / someday
- Pin to the gaps between dots.

## Lost context
- Patrick's original evening of dictation about the doodle board (around May 2026) was not saved
  in the repo or in any Claude Code session. Assume it was a claude.ai chat. If it turns up, fold
  it into DOODLE_SPEC.md.
